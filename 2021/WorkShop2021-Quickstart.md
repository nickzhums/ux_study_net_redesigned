
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

Understanding Azure Resource Hierarchy And Related Classes
--------------------------------------

We have introduced an object hierarchy in the SDK that mimics the object hierarchy in Azure.
Each resources in the SDK is aware of the scope context it is in and has means to access its contained child resources starting with subscription and resource group. With this you will notice unlike previous version of SDK, we now have a single client type of `AzureResourceManagerClient`, and have reduced redundant parameters that the operation methods take. 

To accomplish this we are introducing 4 standard classes for all resources in Azure, for example, `VirtualMachine`, `Subnet` etc.

* ### `[Resource]Data` class

This represents the data that makes up a given resource.  Typically this is populated from the response data from a service call such as GET and provides details about the underlying resource.
Previously this was represented by a **Model** class.

* ### `[Resource]`Operations class

This represents a service client that is scoped to a **single** particular resource.
You can directly execute all operations on that client without needing to pass in scope
parameters such as subscription id or resource name.

* ### `[Resource]` class

This represents a full resource object which contains a **Data** property exposing the details as a **[Resource]Data** type.
It also has access to all of the operations and like the **[Resource]Operations** object is already scoped
to a specific resource in Azure.

* ### `[Resource]sContainer` class

This represents the operations you can perform on a **collection** of resources belonging to a specific parent resource.
This mainly consists of List or Create operations.
For most things the parent will be a **ResourceGroup**, however each parent / child relationship is represented this way.
Such as a **Subnet** is a child of a **VirtualNetwork** or a **ResourceGroup** is a child of a **Subscription**.

Note on different Resource creation methods

* Construct() method returns a builder object to helps Resource construction. The builder gives you access to either a constructed  `[Resource]` instance or CreateOrUpdate() method calls to make the REST call to Azure to creat the resource. For complex resources, the builder may contain additional Fluent style methods to help completely fill out the `[Resource]` without overwhelming number of parameters on the Construct() method.

* CreateOrUpdate() methods takes in a `[Resource]Data` instance and make the REST call to Azure for creation.


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
