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

3. Create the resources:

  ```
  terraform apply -var projectid=${projectid} -var region=${region} -var zone-1=${zone} -var zone-2=${zone2}
  ```

### Deploy a BOSH Director

1. SSH to the bastion VM you created in the previous step. All SSH commands after this should be run from the VM:

  ```
  gcloud compute ssh bosh-bastion-concourse
  ```

2. Configure `gcloud` to use the correct zone, region, and project:

  ```
  zone=$(curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/zone)
  export zone=${zone##*/}
  export region=${zone%-*}
  gcloud config set compute/zone ${zone}
  gcloud config set compute/region ${region}
  export project_id=`curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/project/project-id`
  ```

3. Explicitly set your secondary zone:

  ```
  export zone2=us-east1-d
  ```

4. Create a **password-less** SSH key:

  ```
  ssh-keygen -t rsa -f ~/.ssh/bosh -C bosh
  ```

5. Run this `export` command to set the full path of the SSH private key you created earlier:

  ```
  export ssh_key_path=$HOME/.ssh/bosh
  ```

6. Navigate to your [project's web console](https://console.cloud.google.com/compute/metadata/sshKeys) and add the new SSH public key by pasting the contents of ~/.ssh/bosh.pub:

  ![](../img/add-ssh.png)

  > **Important:** The username field should auto-populate the value `bosh` after you paste the public key. If it does not, be sure there are no newlines or carriage returns being pasted; the value you paste should be a single line.


7. Confirm that `bosh-init` is installed by querying its version:

  ```
  bosh-init -v
  ```

8. Create and `cd` to a directory:

  ```
  mkdir google-bosh-director
  cd google-bosh-director
  ```

9. Use `vim` or `nano` to create a BOSH Director deployment manifest named [`manifest.yml.erb`](manifest.yml.erb).

10. Fill in the template values of the manifest with your environment variables:
  ```
  erb manifest.yml.erb > manifest.yml
  ```

11. Deploy the new manifest to create a BOSH Director:

  ```
  bosh-init deploy manifest.yml
  ```

12. Target your BOSH environment:

  ```
  bosh target 10.0.0.6
  ```

Your username is `admin` and password is `admin`.



[1]: https://github.com/cloudfoundry-incubator/bosh-google-cpi-release/tree/master/docs/concourse (Deploying Concourse on Google Compute Engine — BOSH Google CPI)
