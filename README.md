# GPU-enabled-vSphere-guest-hangs
I needed somewhere to permanently store this issue I've resolved on three separate occasions, separated by a number of months. The issue occurs when you make some kind of change to your vSphere host that cause your direct-io GPU enabled guest to hang at the Windows logo during boot.  
# So your vSphere guest that uses a Nvidia GPU via direct-io only boots to the initial Windows logo and then hangs.  
   1.  Unregister the VM
   2.  ```ssh root@192.168.1.99; cd /vmfs/volumes/datastore1/YOUR-VM; vi YOUR-VM.vmx;``` Strike **i** and change **virtualHW.version** from **19** to **17**
   3.  Launch the Datastore browser, right click on YOUR-VM.vmx, and register the VM
   4.  Remove the direct-io GPU from the guest's virtual hardware configuration
   5.  Remove CD-DVD drive
   6.  Remove SATA controller (given that the drives are connected to the SCSI interface)
   7.  In vSphere, click Edit → VM Options → Advanced → Edit Configuration. Search for svga. Change **SVGA.PRESENT** to **TRUE**
   8.  Power on the virtual machine
   9.  RDP into the VM. Launch devmgmt.msc. Show hidden devices. Uninstall any non-VMWare display drivers. Enable the VMWare display driver. Shutdown the VM.
   10.  Add the direct-io GPU back to the virtual machine. Power on the virtual machine.
   11.  RDP into the virtual machine and install the GPU drivers (Nvidia/AMD/etc...). Disable the VMWare display driver. Shutdown the VM.
   12.  Change **SVGA.PRESENT** to **FALSE**
   13.  Power on the virtual machine.  

