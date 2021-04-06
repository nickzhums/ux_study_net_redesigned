
Quickstart Tutorial - Resource Management (Preview Libraries)
=============================================================

We are excited to announce that a new set of management libraries are
now in Public Preview. Those packages share a number of new features
such as Azure Identity support, HTTP pipeline, error-handling, etc., and
they also follow the new Azure SDK guidelines which create easy-to-use
APIs that are idiomatic, compatible, and dependable.

You can find the details of those new libraries
[here](https://azure.github.io/azure-sdk/releases/latest/#dotnet).

In this basic quickstart guide, we will walk you through how to
authenticate to Azure using the preview libraries and start interacting
with Azure resources. There are several possible approaches to
authentication. [This document](../Prerequisites.md) illustrates the most common scenario.

Prerequisites
-------------
If you haven't already please follow the prerequisite guide [here](../Prerequisites.md).

Authentication and Creating Resource Management Client
------------------------------------------------------

Now that the environment is setup, all you need to do is to create an
authenticated client. Our default option is to use
**DefaultAzureCredential** and create an **AzureResourceManagerClient**.  Since all management APIs go through the same endpoint
you no longer need to create a new client for every resource type like before.

To authenticate to Azure and create an ARM client, simply do the
following:
```csharp
    using Azure.Identity;
    using Azure.ResourceManager.Core;
    using System;
    ...
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
```
From this code snippet, we showed that in order to interact with Resources, we need to create the top-level client first **AzureResourceManagerClient**

More information and different authentication approaches using Azure
Identity can be found in [this document](https://docs.microsoft.com/dotnet/api/overview/azure/identity-readme?view=azure-dotnet).

Understanding Azure Resource Hierarchy
--------------------------------------

In order to reduce both the number of clients needed to perform common tasks and the amount of redundant parameters that each of those clients take,
we have introduced an object hierarchy in the SDK that mimics the object hierarchy in Azure.
Each resource client in the SDK has methods to access the resource clients of its children that is
already scoped to the proper subscription and resource group.

To accomplish this we are introducing 4 standard types for all resources in Azure.

### [Resource]Data

This represents the data that makes up a given resource.  Typically this is the response data from a service call such as GET and provides details about the underlying resource.
Previously this was represented by a **Model** class.

### [Resource]Operations

This represents a service client that is scoped to a particular resource.
You can directly execute all operations on that client without needing to pass in scope
parameters such as subscription id or resource name.

***Old***
```csharp
    ComputeManagementClient computeClient = new ComputeManagementClient(subscriptionId, new DefaultAzureCredential());
    string rgName = "myRgName";
    string vmName = "myVmName";

    // each method we call needs to have the scope passed in to know which vm to operate on
    await computeClient.StartPowerOff(rgName, vmName).WaitForCompletionAsync();
    await computeClient.StartStart(rgName, vmName).WaitForCompletionAsync();
```

***New***
```csharp
    string rgName = "myRgName";
    string vmName = "myVmName";
    Subscription subscription = armClient.DefaultSubscription;
    ResourceGroup resourceGroup = await subscription.GetResourceGroups().GetAsync(rgName);
    VirtualMachine virtualMachine = await resourceGroup.GetVirtualMachines().GetAsync(vmName);

    // no longer need to pass in scope parameters since the object knows the scope
    await virtualMachine.StartPowerOff().WaitForCompletionAsync();
    await virtualMachine.StartPowerOn().WaitForCompletionAsync();
```

This becomes more pronounced when dealing with list methods and performing operations on the response items.

***Old***
```csharp
    ComputeManagementClient computeClient = new ComputeManagementClient(subscriptionId, new DefaultAzureCredential());
    foreach (VirtualMachine vm in computeClient.VirtualMachines.ListAll())
    {
        VirtualMachineUpdate vmUpdate = new VirtualMachineUpdate();
        vmUpdate.Tags.Add("tagKey", "tagValue");
        // when using a subscription list, I need to parse the resource group from the resource ID in order to execute any operations
        await computeClient.VirtualMachines.StartUpdate(GetResourceGroup(vm.Id), vm.Name, vmUpdate).WaitForCompletionAsync();
    }
```

***New***
```csharp
    Subscription subscription = armClient.DefaultSubscription;

    foreach(VirtualMachine virtualMachine in subscription.ListVirtualMachines())
    {
        // because each object is already scoped I no longer need to pass in those variables
        await virtualMachine.StartAddTag("tagKey", "tagValue").WaitForCompletionAsync();
    }
```

### [Resource]Container

This represents the operations you can perform on a collection of resources belonging to a specific parent resource.
This mainly consists of List or Create operations.
For most things the parent will be a **ResourceGroup**, however each parent / child relationship is represented this way.
Such as a **Subnet** is a child of a **VirtualNetwork** or a **ResourceGroup** is a child of a **Subscription**.

### [Resource]

This represents a full resource object which contains a **Data** property exposing the details as a **[Resource]Data** type.
It also has access to all of the operations and like the **[Resource]Operations** object is already scoped
to a specific resource in Azure.

### Putting it all together

Now that we have seen each of the four concepts, let us look at how they all work together in a common scenario.
Imagine that our company requires all virtual machines to be tagged with the owner and we are tasked with
writing a program to add the tag to any missing virtual machines in a given resource group.

```csharp
    //first we construct our armClient
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());

    //next we get a resource group object
    //resourceGroup is a [Resource] object from above
    ResourceGroup resourceGroup = await armClient.DefaultSubscription.GetResourceGroups().GetAsync("myRgName");

    //next we get the container for the virtual machines
    //vmContainer is a [Resource]Container object from above
    VirtualMachineContainer vmContainer = resourceGroup.GetVirtualMachines();

    //next we loop over all vms in the container
    //each vm is a [Resource] object from above
    await foreach(VirtualMachine vm in vmContainer.ListAsync())
    {
        //we access the [Resource]Data properties from vm.Data
        if(!vm.Data.Tags.ContainsKey("owner"))
        {
            //we can also access all [Resource]Operations from vm since it is already scoped for us
            await vm.StartAddTag("owner", GetOwner()).WaitForCompletionAsync();
        }
    }
```

Example: Managing Resource Groups
--------------------------------------

When you first create your armClient you will want to choose which subscription you are going to work in.  There is a convenient **DefaultSubscription** property which will return
the default subscription configured for your user.
```csharp
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
    Subscription subscription = armClient.DefaultSubscription;
```

This is a scoped operations object, and any operations you perform will be done under that subscription.  From this object you have access to all children via container objects
or you can access individual children by id.
```csharp
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
    Subscription subscription = armClient.DefaultSubscription;
    ResourceGroupContainer rgContainer = subscription.GetResourceGroups();
    ...
    string rgName = "myRgName";
    ResourceGroup resourceGroup = rgContainer.Get(rgName);
```

Using the container object we can perform collection level operations such as list all of the resource groups or create new ones under our subscription

***Create a resource group***

```csharp
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
    Subscription subscription = armClient.DefaultSubscription;
    ResourceGroupContainer rgContainer = subscription.GetResourceGroups();
    
    LocationData location = LocationData.WestUS2;
    string rgName = "myRgName";
    ResourceGroup resourceGroup = await rgContainer.Construct(location).CreateAsync(rgName);
```

***List all resource groups***

```csharp
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
    Subscription subscription = armClient.DefaultSubscription;
    ResourceGroupContainer rgContainer = subscription.GetResourceGroups();
    AsyncPageable<ResourceGroup> response = rgContainer.ListAsync();
    await foreach (ResourceGroup rg in response)
    {
        Console.WriteLine(rg.Data.Name);
    }
```

Using the operation object we can perform entity level operations such as updating existing resource groups or deleting them

***Update a resource group***

```csharp
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
    Subscription subscription = armClient.DefaultSubscription;
    ResourceGroup resourceGroup = subscription.GetResourceGroups().Get(rgName);
    resourceGroup = await rgOperation.StartAddTag("key", "value").WaitForCompletionAsync();
```

***Delete a resource group***

```csharp
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
    Subscription subscription = armClient.DefaultSubscription;
    ResourceGroup resourceGroup = subscription.GetResourceGroups().Get(rgName);
    await resourceGroup.DeleteAsync();
```

Example: Creating a Virtual Network
--------------------------------------

In this example we will be create a VirtualNetwork.  Since the SDK follows the resource
hierarchy in Azure, we will need to do this inside of a ResourceGroup.  We will start by creating
a new resource group like we did above

```csharp
    ArmClient armClient = new ArmClient(new DefaultAzureCredential());
    ResourceGroupContainer rgContainer = armClient.DefaultSubscription.GetResourceGroups();
    ResourceGroup resourceGroup = await rgContainer.Construct(LocationData.WestUS2).CreateAsync(rg);
```

Now that we have a ResourceGroup we will now create our VirtualNetwork.  To do this we will use
a helper method on the container object called Construct(...) which will allow us to create the request
object and then send that to the Create(...) method.

```csharp
    VirtualNetworkContainer vnetContainer = resourceGroup.GetVirtualNetworks();
    VirtualNetwork virtualNetwork = await vnetContainer
        .Construct("10.0.0.0/16", location)
        .CreateAsync("myVnetName");
```

Now that we have a VirtualNetwork we must create at least one Subnet in order to add any VirtualMachines.
Again following the hierarchy in Azure Subnets belong to a VirtualNetwork so that is where we will
get our SubnetContainer instance.  After that we will again take advantage of the Construct(...) helper
to create our Subnet.

```csharp
    string subnetName = "mySubnetName";
    SubnetContainer subnetContainer = virtualNetwork.GetSubnets();
    Subnet subnet = await subnetContainer
        .Construct("10.0.0.0/24")
        .CreateAsync(subnetName);
```


Your task will be
----------

1. [Create an Resource Group and an AvailabilitySet under it](createRgAndAvailabilitySet.md)
2. [Add tags to VirtualMachines](addTagsToAllVms.md)

Need help during UX study session?
----------

-   Please explore the APIs via intellisense first to see what makes most sense to you.
-   If you are stuck, your guide may offer hints or direct instructions if the allocated time is running out.

Contributing
------------

For details on contributing to this repository, see the contributing
guide.

This project welcomes contributions and suggestions. Most contributions
require you to agree to a Contributor License Agreement (CLA) declaring
that you have the right to, and actually do, grant us the rights to use
your contribution. For details, visit <https://cla.microsoft.com>.

When you submit a pull request, a CLA-bot will automatically determine
whether you need to provide a CLA and decorate the PR appropriately
(e.g., label, comment). Simply follow the instructions provided by the
bot. You will only need to do this once across all repositories using
our CLA.

This project has adopted the Microsoft Open Source Code of Conduct. For
more information see the Code of Conduct FAQ or contact
<opencode@microsoft.com> with any additional questions or comments.
