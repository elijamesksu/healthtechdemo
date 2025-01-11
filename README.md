# netsmartdemo
A project showcasing GRUB2, Kubernetes, PostgreSQL, and RHEL networking

## 1. Installed VMWare 
VMWare allows a computers hardware to be divided into multiple computers

## 2. Installed RHEL (Red Hat Enterprise Linux)
Decided to use RHEL due to the fact that it has enterprise grade stability, security, support, and maintenance

## 3. Once setup custimize GRUB2
The GRUB bootloader controls how the system starts up. Here are the steps I followed
WHY CUSTOMIZE? Customizing GRUB enhances flexibility, control, and security during the boot process to fine tune system performance, add recovery options, troubleshoot issues
This could help netsmart by enhancing team collaboration, ensuring faster incident resolution and system updates, and making sure there is compliance with healthcare regulations such as HIPAA(Health Insurance Portability and Accountability Act)

MAKE CUSTOMIZATIONS
Added verbose logging: GRUB_CMDLINE_LINUX = "rhgb quiet loglevel =7"
WHY? To troubleshoot boot issues if youre facing problems with your system during boot and identify hardware or driver issues

- Edit GRUB configuration:
  '''bash
  sudo nano /etc/default/grub

- Regenerate the GRUB configuration to apply changes:
  sudo grub2-mkconfig -o /boot/grub2.confg

- Test GRUB:
  WHY? To be able to check the GRUB menu during boot to see the changes(hold shift on boot)
  sudo reboot

## 4. Modify Initrd using dracut



  
