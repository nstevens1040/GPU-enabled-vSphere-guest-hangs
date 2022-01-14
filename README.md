# GPU-enabled-vSphere-guest-hangs
I needed somewhere to permanently store the solution for this issue that I've resolved on three separate occasions, separated by a number of months.  
### So your direct-io GPU-enabled vSphere guest only boots to the Windows logo and then hangs.  
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

