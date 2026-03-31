## Project Title
Designing Secured Public Access to Azure VMs and controlling its Web Access Using Azure Firewall.
## Objective
The aim of the project is deploy an Azure firewall to control outbound web access of VMs, centralize traffic inspection and leverage firewall policy (DNAT Rule) to configure remotes access to VMs via the public IP of the firewall. 
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
The security configuration starts with the deployment of a resource group
![image]
Followed with the deployment of a virtual network alongside with subnets. The first two subnets are for the management of the firewall, while the last two subnets are workload subnets for the HR and Dev teams. The VMs will be deployed in the workload subnets afterwards.\
![image](VNET.png)
The deployed VMs are shown below, the outbound access of both VMs will afterwards be configured to restrict access to certain websites. For instance, users accessing the VMs in HR-subnet will be able to access the Linkedin website.\  
![image](VMs.png)
The Azure Firewall (SKU: Basic) is deployed in the AzureFirewall subnet. The public and private IP address of the firewall are noted and would be used for further configurations.\
![image](FW.png)
A route table is configured and the HR-subnet and Dev-subnet is associated to it. A UDR is added to the route table which enables the traffic from both subnets to pass through the firewall before reaching the internet.\
![image](RT.png)
A firewall policy is created alongside the deployment of firewall. This policy allows the configuration of Application rule, Network rule and DNAT rule\
![image](fwp.png)
The application Rule is created as shown below. The first rule is applied to the HR-subnet while the other rule is applied to the Dev-subnet. The source of the first rule has the CIDR of the HR-subnet, the rule allow web access to the FQDN. The source of the second rule has the CIDR of the Dev-subnet and the rule also allow access to FQDN stated in the rule.\
![image](apr.png)
The network rule allows the firewall to send DNS request to the external DNS server for the resolving of domain names. The rule is applicable to VMs that are located in the subnet stated in the source address (HR-subnet and Dev-subnet)\
![image](nr.png)
The DNAT rule allows users to connect to the VMs in the subnet through RDP. However, the connections is routed through the firewall. The DNAT rule helps the firewall to identify which VM to send the RDP traffic to. To avoid coflict in RDP traffic, different destination port no is specified for each VM. This rule is configured to allow connection to the RDP from any IP address.
![image](dr.png)
The VMs in both subnets are updated with a primary and secondary DNS address to allow the VMs send DNS request to the external DNS server.
![image](vmdns.png)
![image](vmdns2.png)
## Results (Screenshots)
The image shows the process to accessing the VM in the HR-subnet via RDP.
Recall that this connection was possible with the DNAT rule and the connection to the VM is through the firewall public IP. Hence the firewall public IP and the destination port is specified to enable a connection to the VM (in this case, HR-VM)
![image]\
![image]\
The web browser on the HR-VM is launched and it is used to access the websites specified in the application and the connection was succesful while the connection to other websites were denied.
![image]\
![image]\
![image]\
The process was also repeated for the other VM (Dev-VM)
The image shows the process to accessing the VM in the Dev-subnet via RDP.
Recall that this connection was possible with the DNAT rule and the connection to the VM is through the firewall public IP. Hence the firewall public IP and the destination port is specified to enable a connection to the VM (in this case Dev-VM)
![image]\
![image]\
The web browser on the Dev-VM is launched and it is used to access the websites specified in the application and the connection was succesful while the connection to other websites were denied.
![image]\
![image]\
![image]\
## Findings and Recommendations
Using Azure Firewall to restrict outbound web access and control inbound RDP via DNAT is a valid implementation. However, for stronger security, it would require restricting source IPs in the firewall policies, enabling logging and monitoring with Microsoft Sentinel to detect suapicious traffic and ideally replacing public RDP exposure with Azure Bastion to minimize attack surface.
## Implementation of Recommendations
Deployment of Azure Bastion in subnet within the same Vnet as other subnets to provide secured remote access to VMs without traversing the internet. 
![image]\
Configuration of diagnostic settings on Azure firewall to forward logs to a log analytics workspace integrated with Microsoft Sentinel to analyse traffic across the firewall.
![image]
## New Results

## Conclusion


 
