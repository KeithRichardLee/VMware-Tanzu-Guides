# How to install MinIO object storage server to host Ollama and vLLM models offline

With Tanzu AI Solutions, by default, models are pulled down from the internet during installation eg from [Ollama](https://ollama.com/) and [Hugging Face](https://huggingface.co/). This most likely is not an option in most environments and therefore need a method to store models locally. Tanzu AI Solutions supports offline models. 

While many people/companies already have a means to store objects locally eg NFS, S3, WebDAV etc and serve them over http(s), this guide will cover how to install a MinIO object storage server using BOSH. There are many alternative ways to install a MinIO server but this guide will use BOSH as we already have it running from the Tanzu Platform install. Note, Tanzu Platform has a [MinIO tile](https://support.broadcom.com/group/ecx/productdownloads?subfamily=Minio%20Internal%20Blobstore%20for%20VMware%20Tanzu) for a simplified install but that requires a license from MinIO.  

Note: MinIO BOSH Release is licensed under [GNU AFFERO GENERAL PUBLIC LICENSE](https://www.gnu.org/licenses/agpl-3.0.en.html) 3.0 or later.


## High-level flow
- Prepare env
- Setup BOSH CLI
- Download MinIO BOSH Release
- Prepare install files
- Deploy MinIO BOSH Release
- Update DNS
- Configure MinIO server
- Upload models


## Prepare env
ESXi host with the following spare capacity...
- Compute
  - approx 4 vCPU
- Memory
  - approx 16 GB
- Storage
  - approx 137 GB (thin)

Networking
  - IP Addresses
    - 1x MinIO server
  - DNS
    - 1x DNS record
      - MinIO server eg minio.tanzu.lab
  

## Setup BOSH CLI (if not already done so)
- Install [BOSH CLI](https://bosh.io/docs/cli-v2-install/)
- Download BOSH CA cert
  - Tanzu Operations Manager > Settings > Advanced Options > Download root ca cert
- Retrieve BOSH commandline credentials
  - Tanzu Operations Manager > BOSH Director > Credentials > bosh commandline credentials
- Create alias on your workstation (update the path to where you downloaded the cert to) eg
    ```bash
  alias bosh="BOSH_CLIENT=ops_manager BOSH_CLIENT_SECRET=i1GXRRESqDh9Iddtovvx_qCA2Lqyzvab BOSH_CA_CERT=/home/tanzu/root_ca_certificate BOSH_ENVIRONMENT=10.0.70.11 bosh "
    ```
- Verify bosh alias is working eg
  - `bosh vms`

## Download MinIO BOSH Release
```bash
git clone https://github.com/kinjelom/minio-boshrelease.git
```

## Prepare install files
Create a MinIO vars file
```bash
cd minio-boshrelease/manifest/
vi vars/minio-vars-file.yml
```

Paste the following into the file and modify where needed eg minio_disk_type, minio_root_password, and minio_uri
```bash
minio_instances: 1
minio_storage_class_standard: "EC:0"
minio_vm_type: xlarge
minio_disk_type: 102400

minio_root_user: "root"
minio_root_password: "VMware1!"

minio_uri: minio.tanzu.lab
```

Create a MinIO manifest file
```bash
vi manifest.yml
```

Paste the following into the file and update networks name and azs if needed. No need to change if used the defaults in my install guides and scripts
```bash
---
name: ((deployment_name))

instance_groups:
  - name: minio
    instances: ((minio_instances))
    networks: [ { name: tpcf-network } ]
    azs: [ az1 ]
    vm_type: ((minio_vm_type))
    persistent_disk_type: ((minio_disk_type))
    stemcell: default
    env: { persistent_disk_fs: xfs }
    jobs:
      - name: minio-server
        release: minio
        provides:
          minio-server: { as: minio-link }
        properties:
          credential:
            root_user: ((minio_root_user))
            root_password: ((minio_root_password))
          server_config:
            MINIO_BROWSER_REDIRECT_URL: "https://((minio_uri))"
            MINIO_STORAGE_CLASS_STANDARD: "((minio_storage_class_standard))"

variables:
- name: minio_root_user
  type: password
- name: minio_root_password
  type: password

releases:
  - name: "minio"
    version: "2.3.1+minio.2023-11-01T18-37-25Z"
    url: "https://github.com/kinjelom/minio-boshrelease/releases/download/v2.3.1+minio.2023-11-01T18-37-25Z/minio-boshrelease-2.3.1+minio.2023-11-01T18-37-25Z.tgz"
    sha1: "d8a11a6f2e2468d775d07aab7c3f9b1d7d497e86"

stemcells:
  - alias: default
    os: ubuntu-jammy
    version: latest

update:
  canaries: 1
  max_in_flight: 1
  canary_watch_time: 10000-600000
  update_watch_time: 10000-600000
```


## Deploy MinIO BOSH Release


```bash
source ../src/blobs-versions.env
source ../rel.env

bosh -d minio deploy manifest.yml \
  -v deployment_name="minio" \
  -v minio_version="${REL_VERSION}" \
  --vars-file=vars/minio-vars-file.yml \
  --no-redact --fix
```

## Update DNS
- Retrieve IP of MinIO server
  - `bosh -d minio vms`
- Create/update DNS record using the MinIO uri specified in vars/minio-vars-file.yml and IP retrieved from previous command


## Configure MinIO server
Access MinIO console
- Open a browser to http://\<minio-uri\>:9001
- Login with credentials specified in vars/minio-vars-file.yml 

Create bucket
- Buckets > Create Bucket
  - Bucket name: models
  - Create Bucket

Configure access
- Buckets > models > Anonymous > Add Access Rule
  - Prefix: /
  - Access: readonly
  - Save

## Upload models
Upload models to the models bucket using your client of choice eg [MinIO Client](https://min.io/docs/minio/linux/reference/minio-mc.html) (aka mc), [Cyberduck](https://cyberduck.io/) (use S3 http depricated path style request profile), etc 

Note, the port to read/write to the bucket with your client is 9000 eg http://minio.tanzu.lab:9000

See [here](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform-services/genai-on-tanzu-platform-for-cloud-foundry/10-0/ai-cf/tutorials-offline-model-support.html) to learn more about preparing models for offline use.

## Uninstall MinIO
If you wish to uninstall MinIO server, run the following...
```bash
bosh delete-deployment -d minio
```
