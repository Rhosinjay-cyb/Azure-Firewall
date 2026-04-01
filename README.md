## Project Title
Designing Secured Public Access to Azure VMs and controlling its Web Access Using Azure Firewall.
## Objective
The aim of this project is deploy an Azure firewall to control outbound web access of VMs, centralize traffic inspection and leverage firewall policy (DNAT Rule) to configure remotes access to VMs via the public IP of the firewall. 
## Tools Used
Azure Firewall
## Lab Setup
Deployment of virtual network and multiple subnets\
Deployment of virtual machines\
Deployment of Azure Firewall alongside firewall policy\
Configuration of a route table\
Configuration of a user-defined route (UDR)\
Configuration of firewall policy (Application, Network and DNAT rules)\
Updating the VMs DNS server with external DNS address\
Testing of the Firewall 
## Step Taken (Screenshots)
The security configuration starts with the deployment of a resource group (Project-RG)
![image](rg.png)
Followed with the deployment of a virtual network (Project-Vnet) alongside with subnets. The first two subnets are for the management of the firewall, while the last two subnets are workload subnets for the HR and Dev teams. VMs will be deployed in the workload subnets afterwards to test the deployed firewall.

![image](VNET.png)

The deployed VMs (HR-VM & Dev-VM) are shown below, the outbound access of both VMs will be configured to restrict access to certain websites. Meaning, users accessing the VMs in the HR-subnet will have their web access restricted to web apps used by the department.

![image](VMs.png)

The Azure Firewall (Project-FW) of basic SKU is deployed in the AzureFirewall subnet. The public and private IP address of the firewall are noted and would be used in further configurations.

![image](FW.png)

A route table (Project-RT) is configured and the HR-subnet and Dev-subnet is associated to it. A user-define route (OutboundTraffic) is added to the route table which enables the traffic from both associated subnets to pass through the firewall before reaching the internet.

![image](RT.png)

A firewall policy (Project-FWP) is created alongside the deployment of firewall. This policy allows the configuration of Application rule, Network rule and DNAT rule.

![image](fwp.png)

The application Rule is created as shown below. The first rule is applied to the HR-subnet while the other rule is applied to the Dev-subnet. The source of the first rule is the IP address space of the HR-subnet, the rule allows access to only the FQDN (Linkedin & Coursera) specified in the rule while other connections will be dropped. The source of the second rule is the IP address space of the Dev-subnet and the rule only allows access to FQDN (Github) specified in the rule while other connections will also be dropped.

![image](apr.png)

The network rule allows the firewall to send DNS request to the external DNS servers (209.244.0.3, 209.244.0.4) for the resolving of domains. The rule is applicable to VMs that are located in the subnet specified in the source address (HR-subnet and Dev-subnet)

![image](nnr.png)

The DNAT rule allows users to connect to the VMs in the subnet through RDP. However, the connections is routed through the firewall. The DNAT rule helps the firewall to identify which VM to send the RDP traffic to. To avoid conflict in RDP traffic, different destination port no is specified for each VM (3389, 3390) while the traffic is translated to the standard RDP port (3389). This rule is configured to allow connection to the RDP from any IP address. 

Note: When multiple VMs have their RDP traffic routed through the firewall, their destination port numbers must be distinct, which will require assigning random numbers as port numbers.

![image](dr.png)

The VMs in both subnets are updated with a primary and secondary DNS server to allow the VMs send DNS request to the external DNS servers.

For HR-VM
![image](vmdns2.png)

For Dev-VM
![image](vmdns.png)

## Results (Screenshots)
With the completion of the security configurations. The next task is to test the firewall. This will require logging into the earlier depolyed VMs and then establish a connection with the web browser on the VMs to observe if traffic will be blocked/allowed as specified in the configurations.

The following images show the procedures to connecting to the VM in the HR-subnet via RDP.
Recall that this connection was possible with the DNAT rule, and the connection to the VM is through the firewall public IP. Hence the firewall public IP and the destination port is specified to enable a connection to the VM (in this case, HR-VM)

![image](c2vm1.png)

![image](lg2vm1.png)

![image](entry.png)

The web browser on the HR-VM is launched and it is used to access the web apps allowed in the application rule and the connection was succesful while the connection to other websites were denied.

![image](web1.png)

![image](web2.png)

A resulting denied access or failed connection due to the deployed firewall comes in different variants as shown in the following images.

![image](web3.png)

![image](web4.png)

The same procedures were also repeated for the other VM (Dev-VM)
The following images show the procedures to connecting to the VM in the Dev-subnet via RDP.
Also, recall that this connection was possible with the DNAT rule, and the connection to the VM is through the firewall public IP. Hence the firewall public IP and the destination port is specified to enable a connection to the VM (in this case Dev-VM)

![image](c2vm2.png)

![image](lg2vm2.png)

![image](entry2.png)

The web browser on the Dev-VM is launched and it is used to access the web apps specified in the application rule and the connection was succesful while other connections were denied.

![image](web11.png)

![image](web12.png)

## Findings and Recommendations
Using Azure Firewall to restrict outbound web access and control inbound RDP via DNAT is a valid implementation. However, for stronger security, it would require restricting source IPs in the firewall policies, enabling logging and monitoring with Microsoft Sentinel to detect suapicious traffic and ideally replacing public RDP exposure with Azure Bastion to minimize attack surface.
## Implementation of Recommendations
Deployment of Azure Bastion in subnet within the same Vnet as other subnets to provide secured remote access to VMs without traversing the internet. 
![image]\
Configuration of diagnostic settings on Azure firewall to forward logs to a log analytics workspace integrated with Microsoft Sentinel to analyse traffic across the firewall.
![image]
## New Results

## Conclusion


 
