# BRKOPS-1104 - Model it Up: Building Networks with Cisco Modeling Labs - Attendee Materials

### Physical Topology

![Physical Topology Diagram](/CL2025-brkops1104-topology/Physical%20Topology.png)

Thank you for attending the BRKOPS-1104 session.  This GitHub repository contains the lab topology, diagrams, extracted node configurations, and DNS configurations.  The CML Topology requires CML 2.8.1 or newer.  The lab can be downloaded and imported to your CML server.

## Notes:

The Ubuntu-Services node uses ENS2 for external connectivity to your home network.  The IP address is statically set in the configuration.  There are a couple ways to update this:

First, once the lab is imported, simply change the data in this section with an IP address from the external network you've connected to CML.  Be mindful not to add any spaces as this could break the cloud-config.

```
      ethernets:
        ens2:
        #External LAN
          dhcp4: false
          addresses:
          - 192.168.1.210/24
          routes:
          - to: default
            via: 192.168.1.1
          nameservers:
            addresses:
            - 192.168.1.1
```

Second, if you have DHCP on your external network, simply remove all the data and set dhcp4 to true.
```
      ethernets:
        ens2:
        #External LAN
          dhcp4: true
```

Credentials for all hosts are below.  The enable secret for the firewalls is the password below.  If you utilize the proxy, ASDM can be accessed using the crendtials below:

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

As part of the node configuration, BIND9 is installed on the system and files are generated as part of the cloud-config to deploy DNS.  The DNS folder contains the raw files from the node.  It does not however contain all the finalized configurations.  You can clone this repo to the Ubuntu-Services node and copy the contents to the BIND directory.  Follow the steps below:

```
scp netadmin@<ip-of-ubuntu-services-node>
git clone https://github.com/roguerouter/CiscoLive-BRKOPS-1104-Lab.git
cd CiscoLive-BRKOPS-1104-Lab/dns
sudo chmod bind:bind *
sudo cp * /etc/bind
sudo systemctl restart bind9
```

This will add all the IPv6 addressing and completed DNS entries for hosts in the lab.

## Using Squid Proxy

The Ubuntu-Services node contains a proxy that can be used to access http/https services that run in the lab.  An example use case would be ASDM, to setup ASDM, please review the documentation in the ASDM section below.  Squid installs durings the initial boot of the Ubuntu node, however, by default it is not configured.  To enable squid to handle http/https access to the local network in CML, you need to do the following:

```
vi /etc/squid/squid.conf
```

In the squid.conf file, locate this line, and uncomment it ***(remove the #)***. **on my system it is line 1621)**

**Before**
```
# http_access allow localnet
```
**After**
```
http_access allow localnet
```

Save the file and restart the squid proxy service.

```
sudo systemctl restart squid
```

The proxy server will now allow access to ASDM or other http/https based services.

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

## ASDM access

To enable ASDM access, you need to install the lastest Java 1.8 (u451 as of this ReadMe).  You need to configure your proxy settings to allow ASDM to access the nodes inside CML.  

On Apple/Mac systems: 

- Install Java
- On desktop click "Apple Logo" in upper right corner
- Select System settings
- Search for Proxy (it should show Network and proxies)
- Under proxies, select Secure Web Proxy (HTTPS)
- Set the IP as the external IP of your Ubuntu Host (mine was 192.168.1.210, your IP is likely different)
- Set the Port to 3128
- Launch ASDM, try to connect to the 10.255.255.50 (cml-demo-dc-fw) host with admin/s3cr3tpw1

On Windows, this requires editing the run.bat file located in default at:

```
C:\Program Files (x86)\Cisco Systems\ASDM
```

You need to edit this file, I had to move it to my documents directory to make the change.  In the file, you will see the following line:

```
start javaw.exe -Xms64m -Xmx512m -Dsun.swing.enableImprovedDragGesture=true -Djava.net.useSystemProxies -classpath lzma.jar;jploader.jar;asdm-launcher.jar;retroweaver-rt-2.0.jar;log4j-api-2.17.1.jar;log4j-core-2.17.1.jar;commons-codec-1.17.1.jar;commons-logging-1.3.4.jar com.cisco.launcher.Launcher cert.PEM
```

You need to update this line with your proxy server and port.  Note, in the line below, I've used what was set on my host.  Simply change the proxyHost=(IP address) to the IP of your Ubuntu-services external IP
```
-Dhttps.proxyHost=192.168.1.210 -Dhttps.proxyPort=3128
```

The final output will look like the below, with your IP in place of my settings.
```
start javaw.exe -Xms64m -Xmx512m -Dsun.swing.enableImprovedDragGesture=true -Dhttps.proxyHost=192.168.1.210 -Dhttps.proxyPort=3128  -Djava.net.useSystemProxies -classpath lzma.jar;jploader.jar;asdm-launcher.jar;retroweaver-rt-2.0.jar;log4j-api-2.17.1.jar;log4j-core-2.17.1.jar;commons-codec-1.17.1.jar;commons-logging-1.3.4.jar com.cisco.launcher.Launcher cert.PEM
```

Save the changes, move the file back to C:\Program Files (x86)\Cisco Systems\ASDM (if you moved it).  Launch ASDM, try to connect to the 10.255.255.50 (cml-demo-dc-fw) host with admin/s3cr3tpw1
