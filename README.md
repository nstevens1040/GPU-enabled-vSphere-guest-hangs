## So your direct-io GPU-enabled vSphere guest only boots to the Windows logo and then hangs.  
This is where I am logging the steps I take to resolve this issue as I have yet to find a solution that actually works every single time I run into this.  
### Update: 2024-12-12

This time I took a number of different steps that at least appear to have resolved the issue ... this time.  

   1.  Unregister the VM
   2.  ```ssh root@esxi-prod01; cd /vmfs/volumes/datastore1/YOUR-VM; vi YOUR-VM.vmx;``` Strike **i** and change **virtualHW.version** from **19** to **17**
   3.  Launch the Datastore browser, right click on YOUR-VM.vmx, and register the VM
   4.  Remove the direct-io GPU from the guest's virtual hardware configuration
   5.  Remove CD-DVD drive
   6.  Remove SATA controller (given that the drives are connected to the SCSI interface)
   7.  In vSphere, click Edit → VM Options → Advanced → Edit Configuration. Search for svga. Change **SVGA.PRESENT** to **TRUE**
   8.  Power on the virtual machine
   9.  RDP into the VM. Launch devmgmt.msc. Show hidden devices. Uninstall any non-VMWare display drivers. Enable the VMWare display driver.
   10. Download [Display Driver Uninstaller](https://ftp.nluug.nl/pub/games/PC/guru3d/ddu/%5BGuru3D.com%5D-DDU.zip). Extract the zip file ```[Guru3D.com]-DDU.zip```. Then run the executable ```.\[Guru3D.com]-DDU\DDU v18.0.8.9.exe```. This will extract the software to ```\[Guru3D.com]-DDU\DDU v18.0.8.9\```. It's best practice to run the uninstaller in safe mode. So don't run the uninstaller yet.
   11. Launch **msconfig**, tab over to **Boot** and then check the box entitled **Safe boot** and then check the box next to **Network**. Reboot the VM
   12. Launch the VMWare Remote Console by clicking **Actions** → **Console** → **Launch remote console** in ESXi. Log into Windows once the console connects to the guest VM. It should be in safe mode.
   13. Now run the  **Display Driver Uninstaller** executable ```\[Guru3D.com]-DDU\DDU v18.0.8.9\Display Driver Uninstaller.exe``` as administrator.
   14. Select *Device Type* → **GPU** from the drop down menu and then *Device* → **NVIDIA** from the second drop-down menu. Then click on **Clean and do NOT restart**.
   15. Once the Display Driver Uninstaller has finished close the program and launch **msconfig**. Tab over to **Boot** and then uncheck the box next to **Safe boot**.
   16. Shut down the VM.
   17. Add the direct-io GPU back to the virtual machine.
   18. ```ssh root@esxi-prod01; cd /vmfs/volumes/datastore1/YOUR-VM; vi YOUR-VM.vmx;``` Strike **i** and find ```pciPassthru0```. Underneath the last line that matches ```pciPassthru0``` add the following lines <br/>```pciPassthru0.use64bitMMIO = "TRUE"```<br/>```pciPassthru.64bitMMIOSizeGB = "256"```.
   19. Power on the virtual machine, RDP into it and install the GPU drivers. Disable the VMWare display driver. Shutdown the VM.
   20. Change **SVGA.PRESENT** to **FALSE**
   21. Power on the virtual machine.  

### Update: 2022-10-22
[https://tinkertry.com/vmware-vsphere-esxi-7-gpu-passthrough-ui-bug-workaround](https://tinkertry.com/vmware-vsphere-esxi-7-gpu-passthrough-ui-bug-workaround)  

I'm also finding it necessary to execute the command below in an esxi shell to force the host to release GPU resources.
```sh
esxcli system settings kernel set -s vga -v FALSE
```  

### Update: 2022-07-23
[https://communities.vmware.com/t5/VMware-vSphere-Discussions/Deep-investigation-on-GPU-Passthrough-not-working-anymore-after/m-p/457418/highlight/true#M1687](https://communities.vmware.com/t5/VMware-vSphere-Discussions/Deep-investigation-on-GPU-Passthrough-not-working-anymore-after/m-p/457418/highlight/true#M1687)  

I found that I needed to edit ```/etc/vmware/passthru.map```
Look for
```sh
# NVIDIA
10de  ffff  bridge   false
```
and add 
```sh
10de  1401  d3d0     false
```  
underneath it.

### Update: 2022-01-14  
   1.  Unregister the VM
   2.  ```ssh root@esxi-prod01; cd /vmfs/volumes/datastore1/YOUR-VM; vi YOUR-VM.vmx;``` Strike **i** and change **virtualHW.version** from **19** to **17**
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
