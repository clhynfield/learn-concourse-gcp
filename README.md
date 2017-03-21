# Learn Concourse on GCP

Follow [Deploying Concourse on Google Compute Engine][1]

## Prerequisites

1. Install Google Cloud SDK

  ```shell
  brew cask install gcloud
  ```

2. Install Terraform

  ```shell
  brew install terraform
  ```

### Setup your workstation

1. Set your project ID:

  ```
  export projectid='concourse-162220'
  ```

2. Export your preferred compute region and zone:

  ```
  export region=us-east1
  export zone=us-east1-c
  export zone2=us-east1-d
  ```

3. Configure `gcloud` with a user who is an owner of the project:

  ```
  gcloud auth login
  gcloud config set project ${projectid}
  gcloud config set compute/zone ${zone}
  gcloud config set compute/region ${region}
  ```

4. Create a service account and key:

  ```
  gcloud iam service-accounts create terraform-bosh
  gcloud iam service-accounts keys create /tmp/terraform-bosh.key.json \
      --iam-account terraform-bosh@${projectid}.iam.gserviceaccount.com
  ```

5. Grant the new service account editor access to your project:

  ```
  gcloud projects add-iam-policy-binding ${projectid} \
      --member serviceAccount:terraform-bosh@${projectid}.iam.gserviceaccount.com \
      --role roles/editor
  ```

6. Make your service account's key available in an environment variable to be used by `terraform`:

  ```
  export GOOGLE_CREDENTIALS=$(cat /tmp/terraform-bosh.key.json)
  ```

### Create required infrastructure with Terraform

1. Download [main.tf](main.tf) and [concourse.tf](concourse.tf) from this repository.

2. In a terminal from the same directory where the 2 `.tf` files are located, view the Terraform execution plan to see the resources that will be created:

  ```
  terraform plan -var projectid=${projectid} -var region=${region} -var zone-1=${zone} -var zone-2=${zone2}
  ```

1. Create the resources:

  ```
  terraform apply -var projectid=${projectid} -var region=${region} -var zone-1=${zone} -var zone-2=${zone2}
  ```


[1]: https://github.com/cloudfoundry-incubator/bosh-google-cpi-release/tree/master/docs/concourse (Deploying Concourse on Google Compute Engine — BOSH Google CPI)
