# OCP on RHV with ansible

This project shows how you can install **the OpenShift Container Platform on Redhat Virtualization** fully automated with **ansible**. The requriments for this to work are a working dhcp server and pre populated DNS names. 


# Pre Requirements
Pre Requirements:
- You need to have a working dhcp server
- You need to have a working pre populated DNS
- You need a golden template for the deployment
- You need to install the following rpms on your ansible control host for the openshift deployment: openshift-ansible-3.7.14-1.git.0.4b35b2d.el7.noarch, openshift-ansible-playbooks-3.7.14-1.git.0.4b35b2d.el7.noarch



# group_vars and vault

Before you can kickoff the install you will have to make sure that you change the key variables for your own environment. You will have to change the following in groups_vars/all/vars and vault: 

- **key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDe+uyKG6qr7Feh7tNQy9tqyOMvK1a0o7JVcOjLI/5XXNlcMuyVINYQNGVxZpkmjeidaroZASDFz0MuVZwxC8dQ7esf0XPc7+izJ4SEpJnQ0CgOrzLZ9lXhgYeq9uYQcGSrb3dQi4Jt9UuN7zQIdPW9otoAM8HTFFVe2rNJG/pd0R+uWqucgszqzGyNeBykQl6NFv4x0SPu1mxZlqukfwKa7oAhgx4SDDSYcNiEYQsvYDzIDUrAb7bQ74+aE6YB0sNHlM8ax04Z2vTV36Us0x29pNHiSndjQalZ+5z5202f2WwOBHAyzKFDFh0NIq7bsMD1BJV0Z86pF0qsBceLSCf root@localhost.localdomain"**
- **db_size**
-  **db_vol_name**
-  **template_name**
-  **datastore**
- **cluster**
-  **rhvm_addr: "{{ vault_rhvm_addr }}"**
-  **rhv_user: "{{ vault_rhv_user }}"**
- **rhv_pass: "{{ vault_rhv_pass }}"**
- **network_name: ovirtmgmt**
- **rhn_user: "{{ vault_rhn_user }}"**
- **rhn_pass: "{{ vault_rhn_pass }}"**
- **rhn_pool: 8a85f98660c55a380160c2fa572d302b**
- **openshift_master_default_subdomain: apps.local.redhat-demo.com**

All other variables are optional and can be left as a default. 

## Edit the ocp-deploy-rhv.yml

The main file for blueprints is ocp-deploy-rhv.yml. This file includes the definition for **masters**, **nodes**, **lb**, and **nfs** servers.

The following variables in **bolt** are mandatory for each type: <br>
hosts: localhost<br>
  &nbsp;&nbsp;connection: local<br>
 &nbsp;&nbsp;gather_facts: no <br>
  &nbsp;&nbsp;remote_user: root <br>
  &nbsp;&nbsp;vars: <br>
    &nbsp;&nbsp;&nbsp;&nbsp;vm_name: ocp-infranode <br>
    &nbsp;&nbsp;&nbsp;&nbsp;type: infranode <br>
    &nbsp;&nbsp;&nbsp;&nbsp;**infranode: true** <br>
   &nbsp;&nbsp;&nbsp;&nbsp; ipaddr: "{{ node1 }}" <br>
    &nbsp;&nbsp;&nbsp;&nbsp;count: 2 <br>
   &nbsp;&nbsp;&nbsp;&nbsp; **createvm: true** <br>
  &nbsp;&nbsp;roles: <br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- {role: 'ocp-rhv'} <br>

## Important ... If you don't build an HA install you won't need the LB settings
openshift_master_cluster_method: native <br>
openshift_master_cluster_hostname: "{{ hostvars[groups['lb'][0]]['fqdn'] }}" <br>
openshift_master_cluster_public_hostname: "{{ hostvars[groups['lb'][0]]['fqdn'] }}" <br>

## Running the playbook

ansible-playbook ocp-deploy-rhv.yml --vault-pass=vaultpass
