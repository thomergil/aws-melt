# aws-melt

A tool for cleaning up AWS Glacier vaults by retrieving inventories and deleting archives.

## Overview

`aws-melt` automates the process of:

1. Retrieving inventory from Glacier vaults
2. Waiting for inventory jobs to complete
3. Downloading inventory data
4. Deleting all archives in the vaults

## Prerequisites

- Ruby
- jq
- AWS CLI configured with appropriate credentials
- Environment variables:
  - `AWS_ACCOUNT_ID`
  - `AWS_REGION`

## Installation

```bash
git clone https://github.com/thomergil/aws-melt
# brew install jq
# or
# apt-get install jq
cd aws-melt
chmod +x aws-melt
```

## Usage

### List of vaults

You can create `vaults.txt` using the AWS CLI:

```bash
export AWS_ACCOUNT_ID="your-account-id"
export AWS_REGION="your-region"
aws glacier list-vaults --account-id $AWS_ACCOUNT_ID --region $AWS_REGION | jq -r '.VaultList[].VaultName'
```

Then remove any vaults from `vaults.txt` you do not want to delete. 

```
myvault1
myvault2_backup
another_vault
```

**!!Any vault that remains in `vault.txt` will be destroyed!!**

### Basic usage

```bash
export AWS_ACCOUNT_ID="your-account-id"
export AWS_REGION="your-region"
./aws-melt
```

### Resuming deletion

If you have already retrieved the vault inventories (JSON files), you can skip the inventory retrieval process and just perform deletions:

```bash
./aws-melt --delete-only
```

## How it works

1. Reads vault names from `vaults.txt`
2. For each vault:
   - Initiates an inventory-retrieval job
   - Monitors job completion (checking every hour)
   - Downloads inventory when ready
   - Deletes all archives listed in the inventory
3. Creates JSON files named: `{vaultname}_{jobid}_{timestamp}.json`

The `--delete-only` option allows you to restart the deletion process using existing inventory JSON files if the process was interrupted.

## Notes

- Glacier inventory retrieval typically takes 3-5 hours
- Archive deletion is performed one archive at a time
- Large vaults with many archives may take significant time to process

## License

MIT
