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

The external connector for physical hardware has been removed from the final lab.  The network gear supporting it is still there and the node has been replaced with an alpine node.  To recreate the external hardware portion on your own, your CML host needs an additional bridge connector.  The details for configuring it are below.



## Diagrams

Diagrams documenting the physical topology, IPv4 and IPv6 addressing, and IPsec tunnel, VLANS/VNI configuration for the lab are all documented in the CL2025-brkops1104-topology.pdf document.  A topology folder containing the individual diagrams in PDF and PNG format is also available.

## DNS

As part of the node configuration, BIND9 is installed on the system and files are generated as part of the cloud-config to deploy DNS.  The DNS folder contains the raw files from the node.  In the future, these files will be pulled down from the git repo and deployed to the CML Ubuntu-Services node as part of the cloud-init.
