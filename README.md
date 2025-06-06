# BRKOPS-1104 - Model it Up: Building Networks with Cisco Modeling Labs - Attendee Materials

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

## Diagram

![Physical Topology Diagram]([./CL2025-brkops1104-topology/Physical Topology.png](https://github.com/StayFresh-NetworkBytes/CiscoLive-BRKOPS-1104-Lab/blob/main/CL2025-brkops1104-topology/Physical%20Topology.png )
