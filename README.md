# Ext4-LiveUSB
Hi all,
I make a Live persistence on a hdd partition. <br/>
It is actually Raw on FAT32 for Live part and EXT4 for persistence data part.<br/>
I think convert from FAT32 to EXT4 for Live part would make it run faster.<br/>
EXT4 is faster than FAT32 right, although most of the base system is actually on squashfs?<br/>
So I convert Live FAT32 to EXT4.<br/>
Now system not boot at all, only to Grub rescue, where it can not recognize EXT4 partition.<br/>
I digging the google, also come across this thread with no help https://ubuntuforums.org/showthread.php?t=1380886&p=8666605#post8666605<br/>
I digging the Grub rescue prompt and discover that bootx64.efi not load ext4 driver.<br/>
I try to patch the bootx64.efi and lead to these step to the success:

1. Recreate memdisk and config file
```
$mkdir memdisk

$nano memdisk/boot/grub/grub.cfg
            search --file --set=root /.disk/info
            set prefix=($root)/boot/grub
            source $prefix/x86_64-efi/grub.cfg

$tar cvf memdisk.tar memdisk/*
```
2. Recreate bootx64.efi with EXT4 support
```
$nano embeded.cfg
             insmod normal
             set root=(memdisk)
             set prefix=($root)/boot/grub
             source $prefix/grub.cfg

$grub-mkimage  -o bootx64.efi -O x86_64-efi  fat  iso9660 part_gpt part_msdos  \
             normal boot linux configfile loopback chain  efifwsetup efi_gop efi_uga \
             ls search search_label search_fs_uuid search_fs_file  gfxterm gfxterm_background \
             gfxterm_menu test all_video loadenv memdisk ext2 tar  -m memdisk.tar -c embeded.cfg
```
3. copy the bootx64.efi to your /efi/boot folder<br/>
Voila your EXT4 Live partition now boot as if a RAW iso image<br/>

I include the created bootx64 herein so you can just copy it to you ESP/EFI and boot any ext4 LiveUSB/CD

Regards
