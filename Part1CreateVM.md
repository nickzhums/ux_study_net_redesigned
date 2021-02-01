# Create an Azure Virtual Machine
## Goal
Your company has entrusted you with creating their first Virtual Machine in Azure from scratch. For this task you are given a blank subscription with no infrastructure at all.

For reference on general information about the SDK, please see the [Quick Start Guide](QuickStart.md).

## Task description
In Azure, all resources are grouped in Resource Groups, which allows the users to keep track of which application uses what resources more efficiently. The infrastructure and components needed for a Virtual Machine, and the Virtual Machine itself, must be inside a Resource Group.

This task is subdivided in four sections: 
- Creating a client and authenticate to Azure
- Creating the network infrastructure first 
- Create components needed for the Virtual Machine
- Create the virtual machine

### Creating a client and authenticate to Azure

#### Task 1 - Authentication

First step is always to authenticate to Azure and create a client to manage resources, please follow the documentation and create your client to access Azure resources.

Print out the information of the default Azure subscription.

### Setting up the Network Infrastructure
For you to succeed in this task, you will first need to implement the network infrastructure needed to create a Virtual Machine:

- **[Virtual Networks](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)**: A Virtual Network that connects to the internet where all Subnets live.

- **[Subnets](https://docs.microsoft.com/en-us/azure/virtual-machines/network-overview)**: A Subnet is a range of IP addresses where Virtual Machines live.

- **[Network Security Groups](https://docs.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)**: Structure that has different network security rules to filter network traffic, such as open a specific port.

#### Task 2 - Create a resource group
First, from your subscription you need to create the Resource Group where all the resources are going to live. Feel free to name the Resource Group however you want but take into account [naming rules and restrictions for Azure Resources](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules). You are also required to provide a location for the Resource Group, for this task, you can use `"westus2"`.

#### Task 3 - Create a virtual network

Next, you need to create a Virtual Network from the Resource Group, after which you need to add a Subnet in it. When creating the Virtual Network, you can use `10.0.0.0/16` when asked for the CIDR. After that, from that Virtual Network you can create a Subnet where you will use `10.0.0.0/24` for the Subnet CIDR.

#### Task 4 - Create a network security group

Now, it is time to create a Network Security Group (NSG) from the Resource Group. Construct it with a name you like and specify `80` for the port. Opening port 80 in your virtual network will allow HTTP connections from the internet to your Virtual Machine.

### Setting up the Virtual Machine components

Next, you will be setting up all the components needed for a Virtual Machine:
- **[Availability Sets](https://docs.microsoft.com/en-us/azure/virtual-machines/availability#availability-sets)**: Virtual Machines uses this structure to provide redundancy and availability.

- **[Network Interfaces](https://docs.microsoft.com/en-us/azure/virtual-machines/network-overview#network-interfaces)**: A network interface is the interconnection between a Virtual Machine and a Virtual Network. 

- **[Public IP Addresses](https://docs.microsoft.com/en-us/azure/virtual-network/public-ip-addresses#:~:text=Public%20IP%20addresses%20enable%20Azure,IP%20assigned%20can%20communicate%20outbound.)**: Public IP addresses enable Azure resources, in this case, the Network Interface, to communicate to Internet.

#### Task 5 - Create an availability set

Starting with the components, you should create an Availability Set from the Resource Group. You will be asked to provide an SKU Name and a resource name. For this task, you can use `"Aligned"` for the SKU Name and any name you want for the resource name.

#### Task 6 - Create a network interface

Now it is time to setup the Network Interface for the Virtual Machine, but first you need to get a new public IP address. Create a new IP address from the Resource Group and name it as you wish. After that, you should be able to create a Network Interface with the public IP address that you just created.

#### Task 7 - Create a Virtual Machine

For the last step, you will be able to create a new Virtual Machine. Construct the Virtual Machine using the name of the Virtual Machine, username, and password that you want, as well as the Network Interface ID and the Availability Set data.

### Validating
Finally, to validate that your Virtual Machine has been successfully created, display the Virtual Machine ID in the console.

At this point you just successfully created a new Virtual Machine as well as the whole networking infrastructure that is needed for it and any future Virtual Machine that your company will require. Well done.

## Additional references
[Azure REST API references for Compute](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachines/createorupdate)

[Quick Start Guide](QuickStart.md).
