# Shutdown an Azure VM

## Goal

Your company is trying to cut cost in these turbulent times. We noticed no one is using the test Virtual Machines after 9 pm in the evening to 9 am the next morning. Your company's Finance Team advises shutting down these Virtual Machines after the developers go home each night and then restarting them in the morning to save cost. 

For reference on general information about the SDK, please see the [Quick Start Guide](QuickStart.md).

### Task 8 - Description

For this task, write a script that would power off all of the test Virtual Machines, and a separate script that would restart them.

### Identify the Virtual Machines to ShutDown

You only want to shut down the Virtual Machines in the test environment, so let's first identify these Virtual Machines. We have created a ResourceGroup for this workshop task named "ShutdownVM". You can find this Resource Group in your default Subscription.

### Writing the Shutdown Script

Now that we have identified the test environment Virtual Machines, you can power them off. As you shut each VM down, print out a message with the VM name saying it has been shutdown.

### Validating that the Virtual Machines Have Been ShutDown

You know the call was successful if you do not get an exception. For this workshop, we would like you to confirm that the Virtual Machine has been shutdown by getting the latest state from Azure and looking at the InstanceView property. The status code should be "PowerState/stopped".

### Writing the Restarting Script

Similarly, restart all the Virtual Machines and print out a message saying the Virtual Machines have been restarted.

### Validating that the Virtual Machines Have Been Restarted

You know the call was successful if you do not get an exception. For this workshop, we would like you to confirm that the Virtual Machine has been restarted by getting the latest state from Azure and looking at the InstanceView property. The status code should be "PowerState/running".

You have successfully completed this task!

## Additional references
[Azure REST API references for Compute](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachines/createorupdate)

[Quick Start Guide](QuickStart.md).
