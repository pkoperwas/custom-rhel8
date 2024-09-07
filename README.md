# Custom RHEL8 ISO (via kickstart file) 
*steps tested on rhel8*
#


**Download files:**
```
wget https://raw.githubusercontent.com/pkoperwas/custom-rhel8/main/rhel8-ks.cfg -O /nfs-server/ks.cfg
wget (from RedHat customer portal) rhel-8.10-x86_64-dvd.iso -O /nfs-server/rhel-8.10-x86_64-dvd.iso
```

**Prepare custom ISO on your linux**
```
mount -o loop /nfs-server/rhel-8.10-x86_64-dvd.iso /mnt
shopt -s dotglob
mkdir /nfs-server/rhel8iso
cp -pvRf /mnt/* /nfs-server/rhel8iso
umount /mnt
```

**Make a Note of the ISO’s Label**  currently for rhel-8.10-x86_64-dvd.iso is LABEL="RHEL-8-10-0-BaseOS-x86_64"
```
blkid /nfs-server/rhel-8.10-x86_64-dvd.iso
```

**Copy Kickstart File Into ISO Directory**
```
cp /nfs-server/rhel8-ks.cfg /nfs-server/rhel8iso/ks.cfg
```

**Modify the isolinux.cfg file to Point to the New Kickstart File**
```
sed -i '/^\s*append initrd=/ s/$/ inst.ks=cdrom:\/ks.cfg/' /nfs-server/rhel8iso/isolinux/isolinux.cfg
sed -i '/^[[:space:]]*kernel @KERNELPATH@ @ROOT@ quiet/s/$/ inst.ks=cdrom:\/ks.cfg/' /nfs-server/rhel8iso/isolinux/grub.conf
sed -i 's/^default=1/default=0/' /nfs-server/rhel8iso/isolinux/grub.conf
sed -i 's/^timeout 60/timeout 1/' /nfs-server/rhel8iso/isolinux/grub.conf
sed -i 's/set default="1"/set default="0"/' /nfs-server/rhel8iso/EFI/BOOT/grub.cfg
sed -i 's/set timeout=60/set timeout=1/' /nfs-server/rhel8iso/EFI/BOOT/grub.cfg
sed -i '/^[[:space:]]*linuxefi \/images\/pxeboot\/vmlinuz inst.stage2=hd:LABEL=RHEL-8-10-0-BaseOS-x86_64 quiet/s/$/ inst.ks=cdrom:\/ks.cfg/' /nfs-server/rhel8iso/EFI/BOOT/grub.cfg

```

**Generate the New ISO**
```
dnf install genisoimage -y
mkisofs \
-o /nfs-server/custom-rhel8.iso \
-b isolinux/isolinux.bin \
-J -joliet-long -R -l -v \
-c isolinux/boot.cat \
-no-emul-boot \
-boot-load-size 4 \
-boot-info-table \
-eltorito-alt-boot \
-e images/efiboot.img \
-no-emul-boot \
-graft-points \
-V "RHEL-8-10-0-BaseOS-x86_64" \
-jcharset utf-8 /nfs-server/rhel8iso
```

**Make the ISO UEFI Bootable**
```
dnf install syslinux -y
isohybrid --uefi /nfs-server/custom-rhel8.iso
```

**Implant Checksum for New ISO**
```
dnf install isomd5sum -y
implantisomd5 /nfs-server/custom-rhel8.iso
```

