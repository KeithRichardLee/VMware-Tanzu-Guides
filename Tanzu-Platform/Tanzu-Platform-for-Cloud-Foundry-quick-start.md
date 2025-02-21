# Tanzu Platform for Cloud Foundry Quick Start

A guide on how to quickly get up and running with [VMware Tanzu Platform for Cloud Foundry](https://www.vmware.com/products/app-platform/runtimes#cloud-foundry) with minimum resources on VMware vSphere.

Coming soon: a script to perform most of below :raised_hands:

## High-level flow
- Prepare env
- Download bits
- Deploy VMware Tanzu Operations Manager (aka Ops Man)
- Deploy BOSH Director for vSphere
- Deploy Small Footprint Tanzu Platform for Cloud Foundry (aka tPCF)
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


Networking
- IP addresses
  - A subnet with approximatly 10 free IP addresses
    - 1x Ops Man
    - 1x BOSH Director
    - 5x TPCF (Gorouter, blobstore, compute, control, database)
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

Installation Dashboard > Review Pending changes > Apply Changes

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

Installation Dashboard > Review Pending changes > Apply Changes

Congratualations you now have installed and configured Tanzu Platform for Cloud Foundry. Let's go see it in action!


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
      - `cf create-org tanzu-demos-org`
    - Create a Space eg
      - `cf create-space demos-space -o tanzu-demos-org`
    - Target an Org and Space eg
      - `cf target -o tanzu-demos-org -s demos-space`
- Deploy a sample app
  - 


 
## Optional tasks



## Learn more


## Appendix
Above steps were validated against the following...
- VMware vCenter 8.0.3
- VMware ESXi 8.0.3
- VMware Tanzu Operations Manager 3.0.37
- Small Footprint Tanzu Platform for Cloud Foundry 10.0.2
