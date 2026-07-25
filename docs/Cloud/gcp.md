# GCP Cloud SDK CLI Installation & Setup

The Google Cloud SDK contains tools like `gcloud`, `gsutil`, and `bq` command-line tools to interact with Google Cloud Platform services.

---

## Installation on Linux (Debian/Ubuntu)

### Step 1: Add the Google Cloud public key
```bash
sudo apt-get install apt-transport-https ca-certificates gnupg curl -y
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
```

### Step 2: Add the Cloud SDK distribution URI
```bash
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
```

### Step 3: Install the SDK
```bash
sudo apt-get update && sudo apt-get install google-cloud-cli -y
```

---

## Initialization & Setup

Once installed, run the initialization wizard to authenticate and configure your default project:

```bash
gcloud init
```

Follow the prompts to:
1. Log in using your Google account (a browser window will open).
2. Choose/create a default GCP project ID.
3. Configure your default Compute Engine zone (optional).

### Useful Configurations Commands
```bash
# Check current active configuration
gcloud config list

# Set current active project ID
gcloud config set project [PROJECT_ID]

# View active auth account
gcloud auth list
```
