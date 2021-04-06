# Tag an Azure VM

## Goal

You are trying to make sure you can track ownership of all VirtualMachines in your environment.
Your compliance team has identified 5 VirtualMachines that were not property tagged.  Your
task will be to add the proper ownership tag to these VirtualMachines so they can be tracked.

For reference on general information about the SDK, please see the [Quick Start Guide](WorkShop2021-Quickstart.md).

### Task 1 - Description

For this task you are given a vms.json file which has the IDs of the VirtualMachines that are
in violation.  For each VirtualMachine in this file add a new tag with **Key:** "Owner" and
**Value:** "TigerTeam".

### Identify the Virtual Machines to Tag

You only want to tag the Virtual Machines that have been flagged by your compliance
team in the vms.json file.  

### Writing the Tag Script

Now that we have identified the Virtual Machines, you can add a tag to each of them.
As you tag each VM, print out a message with the VM name saying it has been tagged.

### Validating that the Virtual Machines Have Been Tagged

You know the call was successful if you do not get an exception.
For this workshop, we would like you to confirm that the Virtual Machine has been updated
with the new tag by getting the latest state from Azure and validating the Tags collection
it returns has the new tag in it.

You have successfully completed this task!

## Additional references
[Azure REST API references for Compute](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachines/createorupdate)

[Quick Start Guide](WorkShop2021-Quickstart.md).
