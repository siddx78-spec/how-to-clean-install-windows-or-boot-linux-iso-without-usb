## this is born at 2am in the night and finished by 5 am , expect a little typos . will fix it within a day.
## this guide is for windows users, though you can just ask ai to recode these commands to linux equivalent and it should work just fine.

---
- shortcut
    - just download A.7z , its literally a pack of my own laptop's efi partition . check what you need from it, skip microsoft and boot folder if you want. then copy  efi folder and grub2.cfg right into the efi partition's root.
    - windows explorer nags you about permissions , use robocopy, or elevated 7zfm. ( elevated 7zfm is bonkers , you can delete system32 files, rename files, create folders, create files , do crazy stuff . and it will do it easily no prompts , no messing with security tab in windows ) 
---

### need secure boot off. though we are not even loading shady efi images
### need bitlocker drive encryption to be completely off. can be done with manage-bde , ask an ai. go to https://duck.ai and ask it.

--- 
# guide starts here 
## download  


- linux mint iso ( optional but recommended ) and create a tiny 5gb ntfs partition at the very end of your disk, place the iso in root of this ntfs partition , name it exactly "mint.iso"
    - you can place mint.iso in any parition , but it creates a problem. if mint.iso is located in d:\ and then you boot the iso. you wont be later able to mount d:\ partition through nemo/gparted for using it . because of casper loopback doohickery. 
- download windows iso  ( I recommend a ltsc iso  `massgrave.dev` or you can do the usual `https://www.microsoft.com/en-in/software-download/windows11` ) 
    - download and extract the contents of the iso file to root of any partition ( D: , partition is what i am using in this guide )
    - if you already have some files in D:\ partition , then just select all files in d:\ and delete them ( lol no, just move them to D:\my , and then extract the iso to D:\  )  
    - then run `ren D:/efi/boot/bootx64.efi D:/efi/boot/mybootx64.efi`  
    - you can actually rename it to anything mybootx64 , yourbootx64 , gangamstyle.efi , anything. as long as the grub2.efi menu entry points to it.  
    - we need to rename it because we dont want grub2 to load the already installed os , instead search for the extracted iso's efi file.
- further you can place a notautounattend.xml generated using https://schneegans.de/windows/unattend-generator/  , and place it at D:\ ,
    - then when booting into windows installer , select language , next, old/legacy installer , shift+f10 , `setup.exe /Unattend:D:\notautounattend.xml /NoReboot` ,
    - when install finishes, run `wpeutil reboot` to reboot. dont just close it. this is the proper way to reboot after finishing install from a noreboot setup
    - this is useful if you want to run things like `del c:\windows\system32\onedrivesetup.exe`
    - also you can run `fsutil 8dot3name set c: 1 && fsutil 8dot3name strip /s /f c:`
    - note that its not C: always. check it using diskpart, list vol , or even notepad ctrl+o
    - another trick : copy out C:\Program Files\7-Zip to D:\myapps\7zfm , now you can run 7zfm gui from winpe, its amazing.
- download more ram and ssd. the crisis is real.  
- download the official grub2 binaries from https://gitlab.freedesktop.org/gnu-grub/grub/-/releases  
- at the time of writing this the latest available is grub-2.16~rc2-for-windows.zip  
- extract it to anywhere  
- I am using `d:\myapps\grub2\bin`. I usually use d:\myapps as a place to store portable apps. we only need to work with "D:\myapps\grub2\bin\grub-mkstandalone.exe"  
- WE NEED A ELEVATED CMD FOR GRUB BINARIES TO WORK

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

note : you can call grub2.cfg anything. in fact you can have multiple grub cfg files , like grub3.cfg , just get into the grub terminal and load it with configfile (hd0,gp1)/grub3.cfg   
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

## note for guys who have dual boot. 
technically for your laptop to recognize the grubx64.efi , it needs to be placed at 
- either A:\efi\boot
- or A:\efi\ubuntu
- but there are reasons why i chose ubuntu.
1. ubuntu is better than linpus. on lenovo and asus bios. a wild grubx64.efi  , palced in \efi\boot\ is recognized as "linpus lite linux" , it works just fine , but you will be like "woah linpus , what a weird name , lin+puss giggity."
2. when windows is the first install on the disk. it populates the \efi\boot folder with its own boot64.efi ... this does not happen if you install linux first. then install windows as 2nd os. then the windows installer keeps all its efi files confined to \efi\microsoft\ folder.

---

