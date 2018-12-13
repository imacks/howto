How to Mod ThinkPad Boot Screen
===============================
Here's a quick rundown on how to mod your boot screen. Tested on ThinkPad T440s.

Boot screen is the logo that greets you once you turn on your computer. It's encoded in the computer's BIOS/UEFI firmware. T440s comes with a "lenovo" logo by default.

Fortunately, ThinkPad comes with a utility for you to customize this logo, which I think is really awesome. Shame on Surface and Zotec!

1. Download the latest [BIOS update utility](https://pcsupport.lenovo.com/us/en/downloads/ds035965).
2. Install the .msi file from step 1.
   - Accept the terms
   - Take note of where the files are extracted to
   - **DO NOT** install the BIOS update utility on the last screen
3. Prepare your image
   - 60kb max
   - `bmp`, `jpg` or `gif`
   - Max height and width shouldn't exceed 40% of the built-in LCD resolution (e.g. image size for 1920x1080 screen is 768x432 or less).
   - Try using indexed image and web optimized palette to reduce your image size.
   - Name your file as `logo.gif`, `logo.jpg` or `logo.bmp`, depending on your image format.
4. Copy your logo file to the path where files are extracted to in step 2.
5. Run the `winuptp` program and choose "Update ThinkPad BIOS".
6. The program should automatically detect a custom startup image and ask if you want to use it. Choose "Yes".
7. Follow the on-screen instructions. You are advised to switch to AC-power while your BIOS is being replaced.
8. Enjoy your new logo once computer reboots :)

Last updated on 4:51 PM 13/12/2018
