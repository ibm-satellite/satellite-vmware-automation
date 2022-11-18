# Satellite deployer on VMWare

This repository contains ansible playbooks to simplify and accelerate creation of Satellite Location with VMWare infrastructure. 

It helps to :
- check requirements
- create automatically hosts for Satellite
- install pre-requisites on hosts
- create Satellite location
- attach hosts to Satellite location

## Prerequisites:
  - install ibmcloud ansible collection: 
    ```
    ansible-galaxy collection install ibm.cloudcollection
    ```
  - install vmware ansible collection: 
    ```
    ansible-galaxy collection install community.vmware
    ```
  - install netcommon ansible collection: 
    ```
    ansible-galaxy collection install ansible.netcommon  
    ```
  - install python requirements for vmware collection: 
    ```
    pip install -r ~/.ansible/collections/ansible_collections/community/vmware/requirements.txt
    ```
  - install python requirements for netcommon collection: 
    ```
    pip install -r ~/.ansible/collections/ansible_collections/ansible/netcommon/requirements.txt
    ```
  - install python requirements for ibmcloud : 
    pip install jinja2
    pip install dnspython

In order to deploy the virtual machines for the Satellite we must have a RHEL 7 template ready to be used by ansible present in the infrastructure. ( see https://blog.inkubate.io/create-a-rhel-7-terraform-template-for-vmware-vsphere/ )
Before using the playbook we need to define the variables. For this purpose a template file is provided in the directory ```group_vars``` named ```all.yml```.

## Information needed for the inventory
  - ### VmWare information:
    - vCenter FQDN
    - vCenter username
    - vCenter password
    - vCenter datacenter
    - vCenter cluster
    - vCenter datastore
    - vCenter network (the ip class defined on this network must be able to contact internet)
    - vCetner template (this template MUST be based on RHEL7)
    - vCenter folder where to place the virtual machines

  - ### RHEL information (needed for the registration of the nodes to access RedHat repositories)
    - redhat username
    - redhat password
    or
    - organization ID
    - activation key

  - ### IBMCLOUD information
    - IBM Cloud api key (must be created in the cloud.ibm.com dashboard)
    - Region (this can be selected from https://cloud.ibm.com/docs/satellite?topic=satellite-sat-regions)
    - Location that manages the cloud (also selected from https://cloud.ibm.com/docs/satellite?topic=satellite-sat-regions, Ex. 'lon' if you chose the 'eu-gb' region)
    - Location name (this location will be created by the playbook)
    - Resource Group name

  - ### Networking information
    - DNS server
    - IP/Netmask
    - Gateway

  - ### Virtual Machine RHEL7 template information
    - Root password

If the virtual machines cannot be countacted from the deployment machine a jumphost to proxy ssh connections is needed (must be provisioned prior to running the deployment).  
This machine must have 2 network interfaces, one in the same network as the virtual machines that will host the Satellite nodes and one on a network that can be contacted from the deployment machine.   
The only purpose of this machine is to proxy ssh connections from the ansible controller to the Satellite nodes.

### IMPORTANT:
  The sensitive information like passwords or ibmcloud api key need to be protected.  
  For that a vault template is provided ``group_vars/vault.yml.template``, rename the file to ``vault.yml``, add the required information and then encrypt the vault with the command :
  ```
  ansible-vault encrypt vault.yml
  ```
  A password will be requested.  
  The same password will bne needed when you run the playbook.

## How to execute the playbook
  - Go the the directory where the file `` satellite_deploy.yml `` is located
  - Run the command :
    ``` 
    ansible-playbook -i inventory satellite_deploy.yml --ask-vault-pass
    ```