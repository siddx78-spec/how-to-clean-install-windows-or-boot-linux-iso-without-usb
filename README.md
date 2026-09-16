## download

download the official grub2 binaries from https://gitlab.freedesktop.org/gnu-grub/grub/-/releases  
at the time of writing this the latest available is grub-2.16~rc2-for-windows.zip  
extract it to anywhere  
I am using `d:\myapps\grub2\bin`. I usually use d:\myapps as a place to store portable apps. we only need to work with "D:\myapps\grub2\bin\grub-mkstandalone.exe"  
WE NEED A ELEVATED CMD FOR GRUB BINARIES TO WORK
### prepare grub.cfg

press win+R , type `cmd` , press ctrl+shift+enter , click yes on admin elevation prompt.  
run  
`type nul >> d:\myapps\grub2\making\grub.cfg && notepad.exe d:\myapps\grub2\making\grub.cfg`  
paste this , save and exit :  
```
insmod part_gpt
insmod fat
configfile (hd0,gpt1)/grub2.cfg
```
### prepare grub2.cfg  

run  
`type nul >> d:\myapps\grub2\making\grub2.cfg && notepad.exe d:\myapps\grub2\making\grub2.cfg`  
paste this , save and exit :  
```
menuentry "Boot Next Volume" {
    exit
}
menuentry "Windows" {
    insmod part_gpt
    insmod fat
    insmod chain
    insmod ext2
    search --no-floppy --file --set=root /EFI/Microsoft/Boot/bootmgfw.efi
    chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
menuentry "Mint ISO" {
    insmod part_gpt
    insmod fat
    insmod chain
    insmod ext2
    set isofile="/mint.iso"
    search --no-floppy --file --set=root $isofile
    loopback loop $isofile
    linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=$isofile noprompt noeject --
    initrd (loop)/casper/initrd.lz
}
menuentry "Windows Installer" {
    search --no-floppy --file --set=root /efi/boot/mybootx64.efi
    chainloader /efi/boot/mybootx64.efi
}
```
### prepare a batch file  

run  
`type nul >> d:\myapps\grub2\making\test1.bat && notepad.exe d:\myapps\grub2\making\test1.bat`  
paste , save , exit :  
```
@echo off

type nul >> d:\myapps\grub2\making\grub2.cfg && notepad.exe d:\myapps\grub2\making\grub2.cfg
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Get-Partition -DiskNumber 0 -PartitionNumber 1 | Set-Partition -NewDriveLetter A"
mkdir d:\myapps\grub2\making\efi\ubuntu
d:\myapps\grub2\bin\grub-mkstandalone.exe --format x86_64-efi --output=d:\myapps\grub2\making\efi\ubuntu\grubx64.efi "boot/grub/grub.cfg=d:\myapps\grub2\making\grub.cfg"
rem create a backup
robocopy A:\ D:\my\grub2\OriginalEfiBackup\ /E
robocopy D:\myapps\grub2\making\ A:\ /E
echo.
echo.
echo this script is done. mostly.
echo you can further continue this script to remove the assigned letter A:
pause
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Get-Partition -DiskNumber 0 -PartitionNumber 1 | Remove-PartitionAccessPath -AccessPath 'A:\'"
echo. THE END. pausing so that you can look at the execution log.
pause
```
