# Tanzu Platform for Cloud Foundry Geting Started Guide

A guide on how to quickly get up and running with [VMware Tanzu Platform for Cloud Foundry](https://www.vmware.com/products/app-platform/runtimes#cloud-foundry) with minimum resources on VMware vSphere.

You can follow the manual steps below, or use this [script](https://github.com/KeithRichardLee/Tanzu-Platform-for-Cloud-Foundry-automated-install) that has automated all of below :raised_hands:

## High-level flow
- Prepare env
- Download bits
- Deploy VMware Tanzu Operations Manager (aka Ops Man)
- Deploy BOSH Director for vSphere
- Deploy Small Footprint Tanzu Platform for Cloud Foundry (aka tPCF)
- Deploy sample app
- Learn more

## Prepare env
ESXi host (ESXi v7.x or v8.x) with the following spare capacity...
  - Compute: ~18 vCPU, although only uses approx 4 GHz
  - Memory: ~60 GB
  - Storage: ~300GB


Networking
- IP addresses
  - A subnet with approximatly 10 free IP addresses
    - 1x Ops Man
    - 1x BOSH Director
    - 5x TPCF (Gorouter, blobstore, compute, control, database)
    - x various errands, compliations, workers
- DNS service
  - 3 records created
    - 1x Tanzu Operations Manager eg opsman.tanzu.lab
    - 1x TPCF system wildcard eg *.sys.tpcf.tanzu.lab which will resolve to the gorouter IP
    - 1x TPCF apps wildcard eg *.apps.tpcf.tanzu.lab which will resolve to the gorouter IP
- NTP service



## Download bits
- VMware Tanzu Operations Manager
	- https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Tanzu%20Operations%20Manager 
- Small Footprint Tanzu Platform for Cloud Foundry
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
- App Securtiy Groups
  - You are responsible for setting the appropriate ASGs after TPCF finishes deploying: X
- UAA
  - SAML service provider certificate and private key
    - Generate RSA Certificate
      - Domain names: *.apps.tanzu.lab, *.login.sys.tanzu.lab, *.uaa.sys.tanzu.lab, *.sys.tanzu.lab, *.tanzu.lab
- CredHub
  - Internal encryption provider keys
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
  - Tanzu Operations Manager > Tanzu Platform for Cloud Foundry > Credentials > UAA > Admin Credentials
- Create an Org and a Space using either Apps Manager or cf CLI for where we can deploy a sample app
  - Apps Manager
    - See docs [here](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform-for-cloud-foundry/10-0/tpcf/console-login.html) on how to access and use Apps Manager 
  - cf CLI
    - [Install cf CLI](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform-for-cloud-foundry/10-0/tpcf/install-go-cli.html)
    - [Login](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform-for-cloud-foundry/10-0/tpcf/getting-started.html) eg
      - `cf login -a api.sys.tanzu.lab --skip-ssl-validation`
    - Create an Org eg
      - `cf create-org tanzu-demos-org`
    - Create a Space eg
      - `cf create-space demos-space -o tanzu-demos-org`
    - Target an Org and Space eg
      - `cf target -o tanzu-demos-org -s demos-space`
- Deploy a sample app
  - Download spring-music
    - `git clone https://github.com/cloudfoundry-samples/spring-music.git`
  - Build jar file
    - ```
      cd spring-music
      ./gradlew clean assemble
      ```
  - Run app
    - `cf push`
  - Verify app is running, retrieve route, and open app
    - `cf apps` 


## Optional tasks
  - [Deploy Tanzu AI Solutions](/Tanzu-AI-Solutions/Tanzu-AI-Solutions-getting-started-guide.md)


## Learn more


## Appendix
Above steps were validated against the following...
- VMware vCenter 8.0.3
- VMware ESXi 8.0.3
- VMware Tanzu Operations Manager 3.0.37
- Small Footprint Tanzu Platform for Cloud Foundry 10.0.2
