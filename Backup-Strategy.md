# Backup Strategy for Baremetal Kubernetes Cluster

## Goal:

> Create a backup strategy that allows me to quickly and efficiently rebuild my baremetal Kubernetes Cluster

## Key Principles:

1) Use Google Cloud as my backup provider
2) Make the backup as lean as possible
3) Use best practices around securing the backups


### Backup OS Files

1) Create Storage Bucket in Google Cloud

   * Login to [Google Cloud Console](https://console.cloud.google.com)
   * Create a new project for this backup
   * Navigate to Cloud Storage
   * Click on Create Bucket
   * Create a new bucket in the **Nearline** Storage Tier
   * Check box to **Enforce public access prevention on this bucket**
   * Select the **Uniform** option for IAM
   * Under Object Versioning, select **1 Copy** with **30 Day Retention**
   * Click Create
  
2) Create Service Account Principal and Assign Permission to Storage Bucket
  
   * Inside the Cloud Storage Leaf, click on Settings > Interoperability
   * Create Service Account Principal
   * Create an Access Key for your newly created Storage Account
   * Navigate back to Browser > Storage Bucket Name > Permissions
   * Click on Add
   * Select your newly created Service Account under **New Principals**
   * Add the following permissions: **Storage Object Creator**, **Storage Object Viewer**

3) Create New Key for Service Account
   
   * Navigate to IAM and admin tab in Google Cloud Console
   * Click on Service accounts > *Your Service Account* > Keys
   * Create new key and select JSON format
   * Download new key and store it securely

4) Server config
   
   * Copy JSON file to each Server in your cluster
   * Install gsutil using steps [here](https://cloud.google.com/storage/docs/gsutil_install#deb)
   * Create Directory to store .boto file, in this case we will use *~/.config/boto* to store our file.
   * Run the following command to create your boto file, provide path to JSON file when prompted:
> gsutil config -e -o /home/bajadav/.config/boto/.boto

   * Set the following environment variables in you *~/.bash_profile* :
> export BOTO_CONFIG=~/.config/boto/.boto

> export BOTO_PATH=~/.config/boto/.boto

* You should be ready to run gsutil rsync now, here is a sample of the command used to backup */opt* :

> gsutil -m rsync -e -r /opt gs://backup-kubectrl-01/opt &> /var/log/gsutil/opt-`date +\%Y\%m\%d`.log