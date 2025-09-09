# Azure-VNet-Firewall-NSG-and-Bastion-with-HTML-Nginx-Deployment
Beginner level project that explains how to deploy an application on Azure VMs with Networking and use bastion.

## Overview
We will set up a vnet within our resource group in the Azure subscription. Within the vnet there will be three subnets to host the firewall, vm and bastian. vm will have a private ip address hence bastian is needed to connect to the vm in order to install nginx to the vm and create the HTML app. Any user trying to access the app from the internet will have to bypass the firewall first, hence we will use a NAT to translate the firewall's public ip to the VM's private ip.

## Creating a Resource Group
RG is required to make it easier to manage the multiple resources in Azure for this project.
Simply add a name to create a RG.

## Creating a Virtual Network
* Select your created RG under your subscription.
* Named the vnet as gagani-vnet.
* Set region as East US.
* Select enable azure bastian.
* Select create a public ip address for bastian. select standard.
* Enable firewall. Keep the tier to basic. select create new firewall policy and name it as gagani-firewall-policy.
* In the networking section, add the CIDR as 10.0.0.0/16. azure will automatically create 4 subnets namely default, AzureBastianSubnet, AzureFirewallSubnet, AzureFirewallManagement. In the later steps we will use the default subnet to deploy the vm in. You can create more subnets if you require.
* Click on review+create vnet.
* While the deployment is in progress, notice the hierarchy of resource creation. First vnet was created followed by firewall policy and ip addresses for firewall and bastian.
<img width="715" height="833" alt="create a vnet" src="https://github.com/user-attachments/assets/c963ae6b-c287-4aaf-8f26-1f4a5681e7e1" />

##  Creating a Virtual Machine
* Select the same RG.
* Name the vm as nginx-web
* Region same as RG in east us.
* Avaialability zone 1.
* Image is Ubuntu free services eligible
* VM architecture x64
* Size standard B1 free services eligible.
* Enable authentication type as ssh public key.
* Give username as azureuser and select generate ew key pair.
* Allow selected ports in public inbound ports. select ssh 22.
* No changes done to storage or disk configurations.
* In the networking config, select the previously created gagani-vnet.
* Select default subnet.
* Dont enable public ip.
* Select inbound port ssh 22 only.
* Select delete NIC when VM is deleted.
* Click review+create and download the .pem file.
<img width="1260" height="537" alt="create vm" src="https://github.com/userattachments/assets/195fedd5-b287-482a-afa5-8662d0a8db22" />

## Connect to VM through Bastian
Bastian is offered as a service within azure hence you only have to tick the box to enable bastian when creating a vnet. Compared to AWS where you have to create a seperate subnet and vm as the bastian by yourself.

* Go to the created vm and scroll to the option called Connect. Select Bastian.
* Select the authentication type as SSH Private key from local type.
* Give username as azureuser.
* Select the .pem file.
* Click connect.
* Now you are connected to the VM via Bastian.
<img width="1235" height="582" alt="bastian connect" src="https://github.com/user-attachments/assets/75b8585d-f5ff-4320-baff-3d74b10a1d98" />

## VM Configuration
We need to run commands as the root user to install nginx on the vm.
```
sudo su -
```
Update the ubuntu repositories.
```
apt-get update
```
```
apt-get install nginx -y
```
Move to the /var/www/html directory.
```
cd /var/www/html
```
Create an html file with following content.
```
vim index.html
```
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Azure VM Web Project</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f6f8;
      color: #333;
      padding: 40px;
      text-align: center;
    }
    h1 {
      color: #0078D7;
    }
    p {
      max-width: 600px;
      margin: auto;
      font-size: 1.1em;
    }
  </style>
</head>
<body>
  <h1>Welcome to My Azure VM Web Project</h1>
  <p>
    This web page is hosted on a virtual machine deployed in Microsoft Azure.  
    The architecture includes a Virtual Network (VNet), firewall rules, DNAT configuration, and secure access via Bastion.  
    Nginx is used to serve this page, demonstrating basic infrastructure setup and web deployment in a cloud environment.
  </p>
</body>
</html>
```
<img width="1825" height="837" alt="html file" src="https://github.com/user-attachments/assets/0e970337-99c3-4c94-8af7-e8a0a90dc091" />

After saving the file, restart the nginx service.
```
systemctl restart nginx
```
Ensure nginx is running by the following command. It should display the contents of the html file.

## Firewall
For an external user to access this private instance which hosts the webapp, they will have to go through the firewall. 

We have to configure the firewall in such a way that when a user enters <firewall ip>:<port number> on their browser, the firewall should forward the request to the vm.

This is done by configuring the DNAT rules.

* Select your created firewall and select firewall policy.
* In the settings, select DNAT Rules.
* Click add new rule collection.
* Name the rule collection as firewall-nginx-rule.
* Keep rule collection type as DNAT.
* Give priority as 100. (Priority ranges from 100-65,000. Lower numbers = higher priority.)
* Click on add.
<img width="1382" height="492" alt="dnat rule collection" src="https://github.com/user-attachments/assets/2661d30d-5113-4953-a5a3-b98223f7df54" />

* Click on add rule.
* Select the previously created rule collection.
* Give a name as nginx-rule.
* Give your ip as source ip. This is to allow only your device to access the resources.
* Destination ip address is firewall public ip.
* Protocol TCP and Port 4000.
* Give translated type as ip address.
* Give translated address as private ip of vm.
* Port at 80. Request is forwarded to this port.
* Add rule.
<img width="521" height="452" alt="dnat rule1" src="https://github.com/user-attachments/assets/b413cfd7-983c-44f8-ae15-b768fee10484" />
<img width="537" height="682" alt="dnat rule2" src="https://github.com/user-attachments/assets/9a592ad9-1371-4111-a391-992d5f0f1f1c" />

## Access the webpage 
Check whether you can access the webpage by entering <firewall-ip>:4000 on the browser,
If all configurations are correct, the webpage should be displayed.
<img width="1016" height="927" alt="final " src="https://github.com/user-attachments/assets/a8ec913e-4aff-49ff-b020-1746c7d6897c" />

  



