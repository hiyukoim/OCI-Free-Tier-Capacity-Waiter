## OCI Free Tier Capacity Waiter (Docker)

A small Docker Compose setup that **waits for available capacity** in the OCI Free Tier (Ampere A1) and **automatically creates an instance** when capacity becomes available.

### Features

- **Automatic retry** when OCI returns **Out of host capacity**
- **Auto-stop after success** (creates `success.lock`)
- **Free Tier–oriented** settings (Ampere A1)
- **Coolify / Docker compatible**

### Prerequisites

- Docker and Docker Compose installed
- An OCI account with:
  - Tenancy OCID, User OCID, Compartment OCID
  - Subnet ID, Image ID, Availability Domain
  - An API key configured (matching the values in `.env`)

---

### Environment variables (what to set and where to find them)

This project reads configuration from `.env` (or Coolify env variables).  
Here is what each important variable means and **where to find it in the OCI Console**.

- **`OCI_CLI_TENANCY`**
  - **What**: Your tenancy OCID (root of your account).
  - **Where**: OCI Console → top-right profile icon → **Tenancy:<name>** → copy the **OCID**.

- **`OCI_CLI_USER`**
  - **What**: OCID of the user that owns the API key.
  - **Where**: **Identity & Security → Users → (your user)** → copy the **OCID**.

- **`OCI_CLI_FINGERPRINT`**
  - **What**: Fingerprint of the API key you uploaded for that user.
  - **Where**: Same user page → **API Keys** tab → copy the **Fingerprint** of the key you created.

- **`OCI_CLI_REGION`**
  - **What**: Short name of the OCI region, e.g. `ap-tokyo-1`.
  - **Where**: Top-right region selector in the Console (hover to see the region identifier).

- **`COMPARTMENT_ID`**
  - **What**: Compartment where the Ampere A1 instance will be created.
  - **Where**: **Identity & Security → Compartments → (your compartment)** → copy the **OCID**.

- **`AVAILABILITY_DOMAIN`**
  - **What**: Availability Domain string, e.g. `nwkf:AP-TOKYO-1-AD-1`.
  - **Where**: **Compute → Instances → Create instance** 画面で、Availability domain の値をメモするか、既存インスタンスの詳細からコピーします。

- **`SUBNET_ID`**
  - **What**: Subnet OCID where the instance will live.
  - **Where**: **Networking → Virtual Cloud Networks → (your VCN) → Subnets → (target subnet)** → copy the **OCID**.

- **`IMAGE_ID`**
  - **What**: Image OCID for the OS (for example: Ubuntu 22.04 on Ampere A1).
  - **Where**: **Compute → Instances → Create instance** → choose the image, then click the **“OCID”** link next to it, or look it up via the `oci compute image list` CLI.

- **`SHAPE`, `SHAPE_OCPUS`, `SHAPE_MEMORY_GB`**
  - **What**: Instance shape configuration. For Free Tier Ampere A1, a common choice is:
    - `SHAPE=VM.Standard.A1.Flex`
    - `SHAPE_OCPUS=1`
    - `SHAPE_MEMORY_GB=6`
  - **Where**: **Compute → Instances → Create instance** → Shape configuration section.

- **`DISPLAY_NAME`**
  - **What**: Human-readable name of the created instance (anything you like).
  - **Where**: Free text when creating an instance (no strict rule).

- **`OCI_API_KEY_PEM`**
  - **What**: The **multiline** PEM-formatted private key that matches `OCI_CLI_FINGERPRINT`.
  - **Where**:
    - Generated locally when creating the API key (you should have saved the `.pem` file).
    - In Docker `.env`, it is stored as:
      ```env
      OCI_API_KEY_PEM=-----BEGIN PRIVATE KEY-----
      (your key...)
      -----END PRIVATE KEY-----
      ```
    - In Coolify, set it as a **multiline secret** using the UI, not as a literal one-line string.

Once all of these are filled in correctly, the waiter can call the OCI APIs and create the instance when capacity becomes available.

---

### Quick Start (Local Docker)

1. **Create `.env` from the example**

   ```bash
   cp .env.example .env
   ```

2. **Edit `.env`**

   - Fill in your **OCI_* IDs** and **compartment / subnet / image** IDs  
   - Paste your **private key** in the `OCI_API_KEY_PEM` multiline variable

3. **Start the waiter**

   ```bash
   docker compose up -d
   ```

4. **Check result**

   - When an instance is successfully created, a file `/data/success.lock` is created
   - After that, restarting the container does nothing (it sees the lock file and exits)

---

### Using with Coolify

- Deploy this repo as a **Docker Compose** app in Coolify
- **In Environment Variables -> Developer Mode**, set the variables as shown in `coolify-env.md`
  - Use your own **tenancy / user / compartment / subnet / image** values
  - Set `OCI_API_KEY_PEM` as a **multiline secret** in the Coolify UI
- Then deploy the service; logs will show the waiter trying until capacity is available.

---

### Logs (startup check)

When the container starts, you may see messages like this:

```log
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: falling back to frontend: Teletype
debconf: (This frontend requires a controlling tty.)
debconf: falling back to frontend: Noninteractive
```

These look scary, but **they are normal inside Docker**:

- `debconf` is just switching to a **non-interactive mode** because there is no TTY in the container.
- As long as the container keeps running and the script continues, **you can ignore these lines**.
- If the container exits immediately or you see repeated OCI error messages that never recover, then it may indicate a real problem.

---

## 日本語版 README

OCI Free Tier (Ampere A1) の空きを待って  
自動でインスタンスを作成する Docker Compose スクリプトです。

### 特徴

- **Out of host capacity** のとき自動リトライ
- 成功すると `success.lock` を作成して **自動停止**
- Free Tier Ampere A1 向けの設定
- **Coolify / Docker 対応**

### 事前準備

- Docker / Docker Compose がインストールされていること
- OCI アカウントの各種 ID
  - Tenancy OCID / User OCID / Compartment OCID
  - Subnet ID / Image ID / Availability Domain
  - API キー（`.env` の設定と一致させる）

---

### 使い方（ローカル Docker）

1. **`.env` を作成**

   ```bash
   cp .env.example .env
   ```

2. **`.env` を編集**

   - `OCI_*` や `COMPARTMENT_ID` / `SUBNET_ID` / `IMAGE_ID` などを自分の環境に合わせて設定  
   - `OCI_API_KEY_PEM` に秘密鍵 (PEM) を複数行で貼り付け

3. **コンテナを起動**

   ```bash
   docker compose up -d
   ```

4. **結果を確認**

   - 成功すると `/data/success.lock` が作られます
   - その後はコンテナを再起動しても、ロックファイルがあるため何も起きません

---

### Coolify での利用

- このリポジトリを Coolify の **Docker Compose アプリ**としてデプロイします
- **Environment Variables** に **Developer Mode**で`coolify-env.md` の値を設定します
  - 実際の Tenancy / User / Compartment / Subnet / Image ID などを入力
  - `OCI_API_KEY_PEM` は Coolify のシークレットとして **複数行** で設定
- デプロイ後、ログに Capacity Waiter の動作状況が表示されます。

---

### Log 起動確認

コンテナ起動時に、次のようなログが出ることがあります:

```log
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: falling back to frontend: Teletype
debconf: (This frontend requires a controlling tty.)
debconf: falling back to frontend: Noninteractive
```

一見エラーに見えますが、**Docker コンテナ内では正常な挙動**です。

- `debconf` が、TTY のないコンテナ環境向けに **非対話モード** に切り替わっているだけです。
- コンテナが動き続けていてスクリプトが進んでいるなら、**このログは無視して大丈夫**です。
- もしコンテナがすぐに終了したり、OCI エラーが延々と繰り返される場合は、本当に問題が起きている可能性があります。

コンテナ起動後は、**数分ほど様子を見て**、落ち着いて動いているか確認してください。