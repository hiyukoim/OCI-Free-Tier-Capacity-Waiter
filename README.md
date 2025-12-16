# OCI Free Tier Capacity Waiter (Docker)

A Docker Compose script that **waits for available capacity** in the OCI Free Tier (Ampere A1) and **automatically creates an instance** when capacity becomes available.

## Features

* Automatically retries when encountering **Out of host capacity**
* Automatically stops after success (`success.lock`)
* Designed specifically for **OCI Free Tier**
* Compatible with **Coolify** and **Docker**

## Usage

```bash
cp .env.example .env
# Edit .env (set your OCI details and private key)
docker compose up -d
```

-- 日本語版 Readme --
OCI Free Tier (Ampere A1) の空きを待って
自動でインスタンスを作成する Docker Compose スクリプトです。

### 特徴

- Out of host capacity を自動リトライ
- 成功したら自動停止（success.lock）
- Free Tier 向け設計
- Coolify / Docker 対応

### 使い方

```bash
cp .env.example .env
# .env を編集（OCI情報と秘密鍵を設定）
docker compose up -d
