## Coolify Environment Variables

Use this when configuring ** Environment Variables  → Developer Mode** in Coolify.  
All values below are **examples** – replace them with your own OCI information and **never commit real secrets**.

```env
# ---- Basic OCI CLI configuration ----
# Region where you are trying to get Free Tier capacity (example: Tokyo)
OCI_CLI_REGION=ap-tokyo-1

# Your tenancy OCID
OCI_CLI_TENANCY=ocid1.tenancy.oc1..your-tenancy-ocid

# The OCI user OCID that owns the API key
OCI_CLI_USER=ocid1.user.oc1..your-user-ocid

# Fingerprint of the uploaded API key
OCI_CLI_FINGERPRINT=aa:bb:cc:dd:ee:ff:00:11:22:33:44:55:66:77:88:99


# ---- Instance placement ----
# Compartment where you want to create the instance
COMPARTMENT_ID=ocid1.compartment.oc1..your-compartment-ocid

# Availability Domain (example value)
AVAILABILITY_DOMAIN=nwkf:AP-TOKYO-1-AD-1

# Subnet where the instance will be created
SUBNET_ID=ocid1.subnet.oc1.ap-tokyo-1.your-subnet-ocid

# Image ID for the OS (example: Ubuntu 22.04 on Ampere A1)
IMAGE_ID=ocid1.image.oc1.ap-tokyo-1.your-image-ocid


# ---- Shape (Ampere A1 Free Tier example) ----
SHAPE=VM.Standard.A1.Flex
SHAPE_OCPUS=1
SHAPE_MEMORY_GB=6

# Display name for the created instance
DISPLAY_NAME=ubuntu-example-instance


# ---- API private key ----
# Your PEM-format OCI API private key.
# In Coolify, set this as a multiline variable Normal Mode (in the UI). Do not paste here in plain text.
OCI_API_KEY_PEM=-----BEGIN PRIVATE KEY-----
your-multiline-private-key
-----END PRIVATE KEY-----
```