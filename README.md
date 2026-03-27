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
Testing of the Firewall
## Step Taken (Screenshots)
The security configuration starts with the deployment of a virtual network alongside with subnets. The first two subnets are for the management of the firewall, while the last two subnets are workload subnets for the HR and Dev teams. The VMs will be deployed in the workload subnets afterwards.\
![image]
The deployed VMs are shown below, the outbound access of both VMs will afterwards be configured to restrict access to certain websites. For instance, users accessing the VMs in HR-subnet will be able to access the Linkedin website.  
![image]
The Azure Firewall (SKU: Basic) is deployed in the AzureFirewall subnet. The public and private IP address of the firewall are noted and would be used for further configurations.
![image]

## Results (Screenshots)

## Findings

## Recommendations
![](error2.PNG)


 
