---
description: CLI for syncing large artifacts to S3
---

# Ersilia Version Control (EOSVC)

EOSVC is a lightweight CLI designed to sync large data artifacts to AWS S3 while keeping your source code clean in Git. Since Git is not designed to handle large files (like model checkpoints or massive datasets), EOSVC acts as the bridge: Git manages the code, EOSVC manages the data.

For more technical details visit [EOSVC repo](https://github.com/ersilia-os/eosvc/tree/main) in github.

### 🚀 Quick Start

#### Installation

Clone and install the package:.

```
git clone https://github.com/ersilia-os/eosvc.git
cd eosvc
pip install -e .
```

#### AWS Credential Setup

You need AWS credentials to upload or download private data. Run the config command to save them securely:

```
eosvc config \
  --access-key-id "YOUR_ID" \
  --secret-access-key "YOUR_SECRET" \
  --region "eu-central-2"
```

💡 To create your AWS Access Keys, log in to the AWS Console and navigate to the IAM (Identity and Access Management) dashboard. Select Users from the left sidebar, click on your specific username, and open the Security credentials tab. Scroll down to the Access keys section and click Create access key. Select Command Line Interface (CLI) as your use case, acknowledge the safety recommendations, and proceed to generate the keys.

#### Access rules configuration

EOSVC automatically detects what kind of project you are working on based on an `access.json` file in your root folder. Create or edit an `access.json` file in your working directory to define which folders are uploaded to S3 and their access level.

For example

```
{
  "data": "public",
  "output": "private"
}
```

* Files inside the data/ folder will be uploaded with public access (Anyone can download, but you need credentials to upload.).
* Files inside the output/ folder will be uploaded with private access (You must have credentials to view, download, or upload)

💡 Add your data folders to `.gitignore` so you don't accidentally push all of data to GitHub!

### 🛠 Common Commands

#### 📤 Uploading Data

Send your local files to the cloud

<pre><code><strong>eosvc upload --data data/my_big_dataset
</strong></code></pre>

#### 📥 Downloading Data

Pull files from the cloud to your local machine:

```
eosvc download --data data/my_big_dataset
```

#### 🔍 Viewing Remote Files

See what is already stored on S3 for this project:

```
eosvc view
```

### ⚠️ Important Safety Note: Access Lock

To prevent accidental data exposure, EOSVC creates a file at `.eosvc/access.lock.json`. If you try to change a folder from `private` to `public` in your `access.json`, EOSVC will block the operation to protect your data.
