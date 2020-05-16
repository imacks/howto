Enable PXE boot on X550-T2 Network cards
===========================
I bought some Intel X550-T2 network cards for Supermicro servers. Unfortuantely, I can't PXE boot off them in EFI mode. After 
some light reading and experimentation, I came up with the following recipe:

Ref. https://www.supermicro.com/support/faqs/faq.cfm?faq=26077

1. Go https://downloadcenter.intel.com/download/29137 and download PREBOOT.exe

2. Open with 7zip and extract APPS folder to a drive (cannot be Windows drive; VHD is fine).

3. Virtual ISO boot Windows on target Supermicro machine. When installer shows hit `SHIFT`+`F10` to bring up terminal.

4. Mount drive with APPS to server: `Virtual Media > Virtual Storage > Device 2 > Logical Drive Type > (?: SATA HD)`

5. Copy content to RAM drive and install driver (assume drive in step 4 mounted to C):

```
md X:\intelnic
copy C:\APPS\BootUtil\Winx64\* X:\intelnic\
cd X:\intelnic
install.bat
```

6. Flash firmware on NIC:

```
cd C:\APPS\BootUtil\Winx64
# show all nics and note port num of nic you want to flash
BOOTUTILW64E.exe -E
# accept take backup when prompted!
BOOTUTILW64E.exe –nic=? –up=efi –file=C:\APPS\BootUtil\BootIMG.FLB
```

If there are 2 ports on the card, you only need to flash 1 of the ports. You can `BOOTUTILW64E.exe -E` again to verify that the 
version has changed.

7. Reboot server: `Power Control` > `Set Power Reset`

8. Enter BIOS and set EFI boot:

```
Advanced > CPU SLOT7 PCI-E 3.0 X16 OPROM > EFI
Advanced > Onboard LAN Option ROM Type > EFI
Advanced > Network Stack Configuration > Network Stack > Enabled
Advanced > Network Stack Configuration > Network Stack > IPv4 PXE Support > Enabled
Boot > Boot mode select > UEFI
Boot > LEGACY to EFI support > disabled
Save & Exit > Save Changes and Reset
```

9. Enter BIOS again:

```
Advanced > (your nic should appear in menu)
Boot > UEFI NETWORK Drive BBS Priorities > (select nics you want to enable PXE)
Boot > Boot Option #?? > (select nic to enable PXE)
Save & Exit > Save Changes and Reset
```

10. Dada!
