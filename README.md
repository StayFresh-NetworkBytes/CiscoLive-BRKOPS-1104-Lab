# BRKOPS-1104 - Model it Up: Building Networks with Cisco Modeling Labs - Attendee Materials

### Physical Topology

![Physical Topology Diagram](/CL2025-brkops1104-topology/Physical%20Topology.png)

Thank you for attending the BRKOPS-1104 session.  This GitHub repository contains the lab topology, diagrams, extracted node configurations, and DNS configurations.  The CML Topology requires CML 2.8.1 or newer.  The lab can be downloaded and imported to your CML server.

## Notes:

Credentials for all hosts are:

```
username: netadmin
password: s3cr3tpw1
```

The NVE interfaces on  cml-demo-lf1,  cml-demo-lf2,  cml-demo-lf3, as well as cml-demo-border need to be shut/no shut to get connectivity working after starting the lab.  This can be done by accessing the console, logging in with the username/password on those nodes and running the commands:

```
configure terminal
interface nve 1
shut
no shut
end
wr
```

It can also be fixed using the Ansible Playbook in the ansible_nve directory.  Please the documentation below to run this script.

The external connector for physical hardware has been removed from the final lab.  The network gear supporting it is still there and the node has been replaced with an alpine node.  To recreate the external hardware portion on your own, your CML host needs an additional bridge connector.  The details for configuring it are below.

## Diagrams

Diagrams documenting the physical topology, IPv4 and IPv6 addressing, and IPsec tunnel, VLANS/VNI configuration for the lab are all documented in the CL2025-brkops1104-topology.pdf document.  A topology folder containing the individual diagrams in PDF and PNG format is also available.

## DNS

As part of the node configuration, BIND9 is installed on the system and files are generated as part of the cloud-config to deploy DNS.  The DNS folder contains the raw files from the node.  In the future, these files will be pulled down from the git repo and deployed to the CML Ubuntu-Services node as part of the cloud-init.

## Ansible Playbook to restart NVE interfaces

As noted above, you need to restart the NVE interfaces to get the control plane and peers communicating.  In an effort to be more automated, I added a playbook that does the following:

- Performs a shutdown on "interface nve 1"
- Performs a no shutdown on "interface nve 1"
- Pauses for 5 seconds to allow the control plane to communicate
- Performs a show nve peers, registers the output to nve_output variable
- Debugs the nve_output variable and accesses the key of stdout_lines and displays the output of the show nve peers command.

The "show nve peers" and "debug" commands have a tag associated with them of "show_nve_peers".  This allows you to run the show and debug commands only to see the status of the nve peers.  I have attached an image below to show you what a healthy nve peering relationship should look like on each node.

![Healthy NVE Peer example](/ansible_nve/healthy_nve_status.png)

### Setup Ansible

The ubuntu node has Python and Pip3, but in order to install ansible you need to add the python3-venv package.  To do this, from the console in CML or by SSH, access the Ubuntu-Services node.

On this Ubuntu-Services node run the following commands:

First 
```
sudo apt install python3-venv -y
```
After installing the python3 venv:
```
cd ~/CiscoLive-BRKOPS-1104-Lab/ansible_nve/
python3 -m venv ./venv
source ./venv/bin/activate
pip3 install ansible ansible-pylibssh
```
This will install ansible and ansible-pylibssh.  Once installed, you can run the playbooks below.

### Running the playbooks

The inventory has the username and password set.  For initial startup, on the ubuntu-services node run:

```
ansible-playbook restart_nve.yml
```

If you wish to review the show nve peers data again, run:

```
ansible-playbook restart_nve.yml --tag show_nve_peers
```
