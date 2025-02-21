# Tanzu AI Solutions Getting Started Guide

A guide on how to quickly get up and running with [VMware Tanzu AI Solutions](https://www.vmware.com/solutions/app-platform/ai) with minimum resources on VMware vSphere.

Coming soon: a script to perform most of below :raised_hands:

## High-level flow
- Prepare env
- Download bits
- Deploy VMware Tanzu Operations Manager (aka Ops Man)
- Deploy BOSH Director for vSphere
- Deploy Small Footprint Tanzu Platform for Cloud Foundry (aka tPCF)
- Deploy VMware Postgres tile
- Deploy GenAI tile
- Deploy sample app
- Learn more

## Prepare env
ESXi host (ESXi v8.x) with the following spare capacity...
- Compute
  - approx 40 vCPU, although only uses approx 5 GHz
- Memory
  - approx 90 GB
- Storage
  - approx 400GB

GPU *
- Nvidia Pascal architecture or later (Turing, Volta, Ampere, Ada Lovelace, Hopper, Blackwell) GPU with as much vram as possible! eg Tesla P100 / P40 / T4 / V100 / A100, RTX 20/30/40/50 series
  - Notes:
    - This guide has the steps for when using a "cheaper" consumer RTX GPU and using PCI passthrough (aka DirectPath I/O). Enterprise/Pro cards use NVIDIA vGPU and NVAIE instead of PCI passthrough. See [docs](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform-services/genai-on-tanzu-platform-for-cloud-foundry/10-0/ai-cf/tutorials-quickstart-vsphere.html) on how to configure the GenAI tile for vGPU/NVAIE and other requirements such as setting up a Nvidia license service.
    - \* While of course GPUs are faster and more efficent in running models compared to CPUs, they are also very expensive! With Tanzu AI Solutions, you can run the models also on a CPU if you don't have a GPU(s).
- ESXi Host BIOS
  - If your GPU is larger than 16 GB, then set PCIE MMIO to "64 Bit". See your hosts/motherboards manual on how to set this in the BIOS.
- vCenter
  - Host > Configure > Hardaware > PCI Devices > All PCI Devices
    - Select your GPU in the list and then click "Toggle passthrough" so that passthrough is enabled
  - Host > Configure > Hardaware > PCI Devices > Passthroug-enabled Devices
    - Select your GPU in the list
    - Record the "Device ID" and "Vendor ID" from the General Informance section. These ID's can be verifed at [Device Hunt](https://devicehunt.com)
  - Host > Configure > Hardaware > Graphics > Edit
    - Change device type to "Shared Direct"

Networking
- IP addresses
  - A subnet with approximatly 20 free IP addresses (and has internet access so can download models from ollama. Airgapped is supported but not covered in this guide. Please see [here](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform-services/genai-on-tanzu-platform-for-cloud-foundry/10-0/ai-cf/tutorials-offline-model-support.html) for offline model support)
    - 1x Ops Man
    - 1x BOSH Director
    - 5x TPCF (Gorouter, blobstore, compute, control, database)
    - 3x Postgres (broker, instances)
    - 4x GenAI (controller, errand, workers)
    - x various errands, compliations, workers
- DNS service
  - ops manager eg opsman.tanzu.lab
  - system wildcard eg *.sys.tanzu.lab which will resolve to the gorouter IP
  - apps wildcard eg *.apps.tanzu.lab which will resolve to the gorouter IP
- NTP service



## Download bits
- VMware Tanzu Operations Manager (~ 6 GB)
	- https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Tanzu%20Operations%20Manager 
- Small Footprint Tanzu Platform for Cloud Foundry (~ 18 GB)
	- https://support.broadcom.com/group/ecx/productdownloads?subfamily=Tanzu%20Platform%20for%20Cloud%20Foundry  
- VMware Postgres for Tanzu (~ 2 GB)
	- https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware+Tanzu+for+Postgres+on+Cloud+Foundry 
- GenAI on Tanzu Platform for Cloud Foundry (~ 7 GB)
	- https://support.broadcom.com/group/ecx/productdownloads?subfamily=GenAI%20on%20Tanzu%20Platform%20for%20Cloud%20Foundry 

## Deploy VMware Tanzu Operations Manager
Offical docs
- https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-operations-manager/3-0/tanzu-ops-manager/vsphere-deploy.html


## Setup VMware Tanzu Operations Manager default authentication and login
Offical docs
- https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-operations-manager/3-0/tanzu-ops-manager/login.html#login-first-time


## Deploy BOSH Director for vSphere
Offical docs
- https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-operations-manager/3-0/tanzu-ops-manager/vsphere-config.html

Minimum config
- vCenter Config
  - Name
  - vCenter Host
  - vCenter Username
  - vCenter Password
  - Datacenter Name
  - Virtual Disk Type: thin
  - Ephemeral Datastore Names
  - Persistent Datastore Names
  - Standard vCenter Networking
  - Use human-readable VM Names: check
- Director Config
  - NTP Servers
- Create Availability Zones
  - Add
    - Name
    - IaaS Configuration
    - Cluster
    - Resource Pool
- Create Networks
  - Add Network (a single network is ok and is used in this guide but best practice is seperate networks for management, TPCF, and services)
    - Name
      - Subnets
        - vSphere Network Name
        - CIDR
        - Reserved IP Ranges
        - DNS
        - Gateway
        - Availability Zones
- Assign AZs and Networks
  - Singleton Availabilty Zone
  - Network
- Security
  - Include Tanzu Ops Manager Root CA in Trusted Certs: check

Installation Dashboard > Review Pending changes > Apply Changes (Installation takes approx 10 mins)

## Deploy Small Footprint Tanzu Platform for Cloud Foundry
Offical docs
- https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform-for-cloud-foundry/10-0/tpcf/add-tas-vms.html

Import the Small Footprint Tanzu Platform for Cloud Foundry tile by clicking "import a product" and then click "+" once its imported

Minimum config
- Assign AZs and Networks
  - Network (if created more than one)
- Domains
  - System domain eg sys.tanzu.lab
  - Apps domain eg apps.tanzu.lab
- Networking
  - Gorouter IPs eg enter a single IP from your network. Your system and apps wildcard domains will resolve to this IP
  - Certificates and private keys for the Gorouter
    - Add
      - Name
      - Generate RSA Certificate
        - Domain names: *.apps.tanzu.lab, *.login.sys.tanzu.lab, *.uaa.sys.tanzu.lab, *.sys.tanzu.lab, *.tanzu.lab
  - TLS termination point: Gorouter
- App Developer Controls
  - Maximum disk quota per app: 6144 (need to bump up from 2048 so can run Open WebUI app)
- App Securtiy Groups
  - You are responsible for setting the appropriate ASGs after TPCF finishes deploying: X
- UAA
  - SAML service provider certificate and private key
    - Generate RSA Certificate
      - Domain names: *.apps.tanzu.lab, *.login.sys.tanzu.lab, *.uaa.sys.tanzu.lab, *.sys.tanzu.lab, *.tanzu.lab
- CredHub
  - Internal encryption provide keys
    - Add 
      - Name
      - Key (must be at least 20 characters long)
      - Primary: check
- Resource Config
  - Backup Restore Node: 0
  - MySQL Monitor: 0

Note: Don't apply changes yet. Proceed to the next section.


## Deploy VMware Postgres for Tanzu Application Service
Offical docs
- https://techdocs.broadcom.com/us/en/vmware-tanzu/data-solutions/tanzu-for-postgres-on-cloud-foundry/10-0/postgres/install.html

Import the VMware Postgres for Tanzu Application Service tile by clicking "import a product" and then click "+" once its imported

Minimum config
- Assign AZs and Networks
  - Network
  - Service Network (select the available network if only created one, otherwise select a dedicated service network if created)
- On-Demand Plans
  - Add
    - AZs to deploy postgres instances of this plan: check

Installation Dashboard > Review Pending changes > Apply Changes


## Deploy GenAI on Tanzu Platform
Offical docs
- https://techdocs.broadcom.com/us/en/vmware-tanzu/platform-services/genai-on-tanzu-platform-for-cloud-foundry/10-0/ai-cf/tutorials-quickstart-vsphere.html

Import the GenAI on Tanzu Platform tile by clicking "import a product" and then click "+" once its imported

Minimum config
- Assign AZs and Networks
  - Network
  - Service Network (select the available network if only created one, otherwise select a dedicated service network if created)
- Database Config
  - Offering Name: postgres
  - Plan Name: on-demand-postgres-db
- Infrastructure Config
  - Custom VM types for vSphere
    - Add
      - Name: eg gpu
      - Processing technology: PCI Passthrough
      - PCI Passthrough Vendor ID: ID in hex recorded in pre-reqs eg 0x10DE
      - PCI Passthough Device ID: ID in hex recorded in pre-reqs eg 0x2803
      - Note: if you GPU is larger than 16GB then also configure the following
        - PCI Passthrough vmx options - use 64bit MMIO: TRUE
        - PCI Passthrough vmx options - size for 64bit MMIO: 64
- Model Config
  - Ollama Models
    - Add
      - Model name: gemma2:2b (or model of choice from ollama.com that can run/fit on your GPU. For example, gemma2:9b requires approx 9 GB vram)
      - Model Capabilities: chat
      - VM Type: name provided in previous infrastructure config step eg gpu
      - Availability Zone: check
    - Add
      - Model name: nomic-embed-text
      - Model Capabilities: Embedding
      - VM Type: cpu
      - Availability Zone: check
   
Installation Dashboard > Review Pending changes > Apply Changes

Congratualations you now have installed and configured Tanzu Platform for Cloud Foundry and Tanzu AI Solutions. Let's go see it in action!


## Deploy a sample app
- Retrieve UAA admin credentials
  - Tanzu Operations Manager > Tanzu Platform for Cloud Foundry > Credentials > UUA > Admin Credentials
- Create an Org and a Space using either Apps Manager or cf CLI for where we can deploy a sample app
  - [Apps Manager ](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform-for-cloud-foundry/10-0/tpcf/console-login.html)
  - cf CLI
    - [Install cf CLI](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform-for-cloud-foundry/10-0/tpcf/install-go-cli.html)
    - [Login](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform-for-cloud-foundry/10-0/tpcf/getting-started.html) eg
      - `cf login -a api.sys.tanzu.lab`
    - Create an Org eg
      - `cf create-org tanzu-ai-solutions-org`
    - Create a Space eg
      - `cf create-space demos-space -o tanzu-ai-solutions-org`
    - Target an Org and Space eg
      - `cf target -o tanzu-ai-solutions-org -s demos-space`
- Deploy a sample app
  - [Spring-Metal](https://github.com/nkuhn-vmw/GenAI-for-TPCF-Samples/tree/main/spring-metal)
  - [Open WebUI](https://github.com/nkuhn-vmw/GenAI-for-TPCF-Samples/tree/main/open-webui-cf)

 
## Optional tasks
- [Agentic AI and MCP (Model Context Protocol) example with Tanzu Platform for Cloud Foundry](/Tanzu-AI-Solutions/Agentic-AI-and-MCP-example-with-Tanzu-Platform-for-Cloud-Foundry.md)
- [Monitor GPU using nvtop](https://github.com/KeithRichardLee/VMware-Tanzu-Guides/blob/main/Tanzu-AI-Solutions/how-to-monitor-GPU-with-nvtop.md)
- [Configure a Healthwatch Grafana dashboard](https://github.com/KeithRichardLee/VMware-Tanzu-Guides/blob/main/Tanzu-AI-Solutions/how-to-add-a-Healthwatch-Grafana-dashboard-for-Tanzu-AI-Solutions.md)


## Learn more
- [VMware Tanzu AI Solutions landing page](https://www.vmware.com/solutions/app-platform/ai)
- Blogs
  - [What is AI Middleware, and Why You Need It to Safely Deliver AI Applications](https://blogs.vmware.com/tanzu/what-is-ai-middleware-and-why-you-need-it/)
  - [Building AI Apps in 2025: Your Brilliant Ideas Deserve a Genius, Future-Proof Platform](https://blogs.vmware.com/tanzu/building-intelligent-apps-in-2025/)
  - [It’s OK to ask why AI prototypes are not getting to production](https://blogs.vmware.com/tanzu/its-ok-to-ask-why-ai-prototypes-are-not-getting-to-production/)
  - [Why Build GenAI Apps the Hard Way? Get an App Platform Instead!](https://blogs.vmware.com/tanzu/why-build-genai-apps-the-hard-way-get-an-app-platform-instead-2/)
  - [Demystifying AI Integration with Cloud Foundry: A Spring AI Deep Dive](https://blogs.vmware.com/tanzu/demystifying-ai-integration-with-cloud-foundry-a-spring-ai-deep-dive/)


## Appendix
Above steps were validated against the following...
- VMware vCenter 8.0.3
- VMware ESXi 8.0.3
- Nvidia RTX 4060 TI 16GB GPU
- VMware Tanzu Operations Manager 3.0.37
- Small Footprint Tanzu Platform for Cloud Foundry 10.0.2
- VMware Postgres for Tanzu Application Service 10.0.0
- GenAI on Tanzu Platform for Cloud Foundry 10.0.2
