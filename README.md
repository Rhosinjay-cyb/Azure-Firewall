## Project Title

Designing Secured Public Access to Azure VMs and controlling its Web Access Using Azure Firewall.

## Objective

The aim of this project is deploy an Azure firewall to control outbound web access of VMs, centralize traffic inspection and leverage firewall policy (DNAT Rule) to configure remotes access to VMs via the public IP of the firewall. 

## Tools Used

Azure Firewall, Azure Bastion, Log Analytics Workspace, DevTools

## Lab Setup
* Creation of a resource group
* Creation of virtual network with multiple subnets
* Deployment of virtual machines
* Deployment of Azure Firewall alongside firewall policy
* Configuration of a route table
* Configuration of a user-defined route (UDR)
* Configuration of firewall policy (Application, Network and DNAT rules)
* Updating the VMs' DNS server with external DNS address
* Testing of the Firewall 

## Step Taken (Screenshots)

The security configuration starts with the creation of a resource group (Project-RG)

![image](rg.PNG)

Followed with the creation of a virtual network (Project-Vnet) alongside with subnets. The first two subnets are for the management of the firewall, while the last two subnets are workload subnets for the HR and Dev teams. VMs will afterwards be deployed in the workload subnets to test the deployed firewall.

![image](VNET.png)

The deployed VMs (HR-VM & Dev-VM) are shown below, the outbound access of both VMs will be configured to restrict access to certain web apps. Meaning, users accessing the VMs in the HR-subnet will have their web access restricted to web apps allowed in the firewall policy. Both VMs are Windows Server 2019 Datacenter Gen2, and they will be accessed via remote desktop protocol (RDP).

![image](VMs.png)

The Azure Firewall (Project-FW) of basic SKU is deployed in the AzureFirewallsubnet. The public and private IP address of the firewall are noted and would be used in further configurations. The firewall policy (Project-FWP) is also created alongside the deployment of the firewall.

![image](FW.png)

A route table (Project-RT) is configured and the HR-subnet and Dev-subnet is associated to it. A user-define route (OutboundTraffic) is added to the route table which enables the traffic from both associated subnets to be routed through the firewall before reaching the internet.

![image](RT.png)

With the firewall policy (Project-FWP), the Application rule, Network rule and DNAT rule would be configured.

![image](fwp.png)

The application Rule is created as shown below. The first rule is applied to the HR-subnet while the other rule is applied to the Dev-subnet. The source of the first rule is the IP address space of the HR-subnet, the rule allows access to only the fully qualified domain name (FQDN) (linkedin.com & coursera.org) specified in the rule while other connections will be blocked. The source of the second rule is the IP address space of the Dev-subnet and the rule only allows access to FQDN (github.com) specified in the rule while other connections will also be blocked.

![image](appr.PNG)

The network rule allows the firewall to send DNS request to the external DNS servers (209.244.0.3, 209.244.0.4) for the resolving of domains. The rule is applicable to VMs that are located in the subnet specified in the source address (HR-subnet and Dev-subnet)

![image](nnr.png)

The DNAT rule allows users to connect to the VMs in the subnet through RDP. However, the connections is routed through the firewall. The DNAT rule helps the firewall to identify which VM to send the RDP traffic to. To avoid conflict in RDP traffic, different destination port no is specified for each VM (3389, 3390) while the traffic is translated to the standard RDP port (3389). This rule is configured to allow connection to the RDP from any IP address. 

Note: When multiple VMs have their RDP traffic routed through the firewall, their destination port numbers must be distinct, which may require assigning random numbers as port numbers. Connecting to the VM will requiring specifying the Public IP of the firewall and the destination port no as a socket.

![image](dr.png)

The VMs in both subnets are updated with a primary and secondary DNS server to allow the VMs send DNS request to the external DNS servers.

For HR-VM
![image](vmdns2.png)

For Dev-VM
![image](vmdns.png)

## Results (Screenshots)

With the completion of the security configurations. The next task is to test the firewall. This will require connecting remotely to the earlier depolyed VMs and then establishling a connection with the web browser on the VMs to observe if traffic will be blocked/allowed as specified in the firewall policy.

The following images show the procedures to connecting remotely to the VM in the HR-subnet via RDP.
Note: Recall that this connection was possible with the DNAT rule, and the connection to the VM is through the firewall public IP. Hence the firewall public IP and the destination port is specified to enable a connection to the VM (in this case, HR-VM)

![image](c2vm1.png)

![image](lg2vm1.png)

![image](entry.png)

The web browser on the HR-VM is launched and it is used to access the web apps allowed in the application rule and the connection was succesful while the connection to other websites were denied.

![image](web1.png)

![image](web2.png)

A resulting denied access or failed connection due to the deployed firewall comes in different variants as shown in the following images.

![image](web3.png)

![image](web4.png)

The same procedures were also repeated for the other VM (Dev-VM).
The following images shows the procedures to connecting to the VM in the Dev-subnet via RDP.
Also, recall that this connection was possible with the DNAT rule, and the connection to the VM is through the firewall public IP. Hence the firewall public IP and the destination port is specified to enable a connection to the VM (in this case Dev-VM)

![image](c2vm2.png)

![image](lg2vm2.png)

![image](entry2.png)

The web browser on the Dev-VM is launched and it is used to access the web app specified in the application rule and the connection was succesful while other connections were denied.

![image](web11.png)

![image](web12.png)

## Findings and Recommendations

The implementation of Azure Firewall to restrict outbound web access and control inbound RDP via DNAT was succesful as shown in the result section. However, it was observed that the web pages load partially. Meaning, the domains that contains the dependencies (logos, images, videos and other web files) of the web page has been blocked by the firewall. Hence, this drawback needs to resolved to make this security solution fit for production standard. Other areas of concern to enable stronger security include restricting source IPs in the firewall policy, enabling diagnostic setting of the firewall to detect suspicious traffic, and replacing public RDP exposure with Azure Bastion to minimize attack surface.

## Implementation of Recommendations

**1.** Starting with analysing the partial loading of web pages. The task was to identify the domain of the dependencies which the firewall blocked access to. Two different methods were used in identifying the FQDNs. The first method was to use the DevTools of the web browser where the web page is being accessed and the other method is the analysis of firewall logs. 

The DevTools was launched by pressing the F12 key on the keyboard while on the affected web page. In the DevTools environment, the dropped FQDNs could be identified from the networks tab after reloading the web page.

![image](EVI21.png)

A close observation of the source tab shows the various files in the domains which the firewall blocked access to causing the disruption of the look of the web page.

![image](EVI3.PNG)

For the other method involving firewall logs, the diagnostic setting of the firewall needs to be enabled for the logs to be accessible.
The diagnostics settings of the firewall was enabled from the monitoring blade of the firewall and the firewall logs were forwarded to a log analytics workspace (Project-workspace) which was created earlier.

![image](FW2LAW.PNG)

Afterwards, the firewall logs particularly the application rule log was analysed to identify the blocked domains

![image](EVI20.png)

Having identified the affected domains (d3njjcbhbojbot.cloudfront.net & coursera_assets.s3.amazonaws.com) with two different methods. The application rule is updated with the newly discovered domains while using a wildcard. 

Note: It is recommended to avoid broad wildcards unless neccessary.

![image](EVI22.png)

The web page was refreshed and it now loads properly

![image](EVI6.PNG)

The application rules allowing web access to other web applications (Linkedin & Github) were equally updated with their respective affected FQDNs and the web apps were refreshed on the web browser.

![image](EVI8.PNG)

![image](evi9.PNG)

**2.** For the deployment of Azure Bastion to provide secured remote access to the VMs without exposing them to the internet. Firstly, AzureBastionSubnet is added to subnets within the virtual network.

![image](BST-SNET.png)

Afterwards, Azure Bastion is deployed.

![image](BST.PNG)

Now connecting remotely to the HR-VM via Azure Bastion

![image](CVB.PNG)

Entering the login details of the HR-VM

![image](ENP.PNG)

Succesful access into the HR-VM via Azure Bastion

![image](ENHR.PNG)

The processes were repeated to connect remotely to the Dev-VM via Azure Bastion.

![image](ENDEV.PNG)

## Conclusion

This project demonstrated how to secure Azure VMs by controlling outbound traffic with Azure Firewall, and leveraging Azure Bastion for secure remote connection to the VMs to reduce attack surface. The project also demonstrate how to improve security solutions, in this case, enhancing the look and feel of the approved web apps by simply indentifying their dependencies and granting access to them via the firewall policy. Additionally, collection of firewall logs and forwarding it to a log analytics workspace provided visibility and monitoring, reflecting a practical layered approach to cloud workload protection.

## Future Works

To plan and implement the integration of Microsoft sentinel with the log analytics workspace (Project-workspace) and then collecting logs from Azure Bastion and the Virtual machines and sending it to the Microsoft sentinel-integrated log analytic workspace. These logs alongside the already enabled firewall logs would aid centralized monitoring and threat detection.


 
