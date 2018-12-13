How to Mod ThinkPad Boot Screen
===============================
Here's a quick rundown on how to mod your boot screen. Tested on ThinkPad E440.

Boot screen is the logo that greets you once you turn on your computer. It's encoded in the computer's BIOS/UEFI firmware. E440 comes with an italic "lenovo" logo by default.

Fortunately, ThinkPad comes with a utility for you to customize this logo, which I think is really awesome. Shame on Surface and Zotec!

1. Download the latest [BIOS update utility](https://download.lenovo.com/pccbbs/mobiles/j9uj28ww.exe). I found it [from here](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-edge-laptops/thinkpad-edge-e440/downloads).
2. Install the .msi file from step 1.
   - Accept the terms
   - Take note of where the files are extracted to
   - **DO NOT** install the BIOS update utility on the last screen
3. Prepare your images. The help in `BOOT_LOGO.txt` is strangely misleading. I figured these out emperically:
   - You need to prepare 2 files: `LOGO1` and `LOGO2`. Despite what they said about OS optimization settings, the truth is `LOGO1` is displayed when secure boot is disabled, and `LOGO2` shows up when I enable secure boot.
   - 30kb max per image. Strictly so!
   - Only `gif` worked for me. `bmp` is too big per size limitation above, and `jpg` resulted in color glitches.
   - E440 has a built-in LCD resolution of 1360x768. I set `LOGO1` resolution to 640x480, and it covered the whole screen. I observed a bit of stretching, and had to modify my picture accordingly. It's trial-and-error.
   - `LOGO2` can support the LCD's resolution, thus I made a picture that is 1360x768 and it worked fine.
   - One gotcha: the lower bottom of the screen is painted over with a black strip in `LOGO1` sometime after boot. No big deal for me.
4. Copy your `LOGO1.gif` and `LOGO2.gif` to the path where files are extracted to in step 2.
5. Make sure you install `.NET 3.5`! It's available in "features and options" in Windows settings.
6. Plug your laptop to AC power. BIOS flash utility will detect and refuse to work on battery.
7. Run the `winuptp` program and choose "Update ThinkPad BIOS".
   - If your BIOS is password protected, it will prompt you for the "supervisor password" along the way.
   - If you are flashing the same BIOS multiple times on the same PC, you need to disable "rollback prevention", also in BIOS.
   - The program should automatically detect a custom startup image and ask if you want to use it. Choose "Yes".
   - Follow the instructions. The program works quickly. I come to a prompt that talks about rebooting. Click "OK" and the program quits.
   - **DO NOT** do anything. Your laptop should reboot by itself.
   - Your laptop reboots into a terminal type of situation, where a console app flashes your ROM and reboots the laptop when done.   
8. Enjoy your new logo once computer reboots :)
