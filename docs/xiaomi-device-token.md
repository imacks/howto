Xiaomi Device Token
===================
This guide is adapted from https://python-miio.readthedocs.io/en/latest/discovery.html.

Xiaomi sells a lot of IoT gadgets for good value, except that you need to use them exclusively with the Mi Home app. The secret is that a lot of 
these gadgets use a simple token based authentication mechanism. This guide shows you how to get the token from a compatible device.

Tested on OnePlus 5 running Android 9 and Yeelight S1 (LYAI003) bought in China.

  1. Install version 5.4.49 of MiHome
    - Xiaomi developers shipped this version with debug log on. This is our way in.
    - apkmirror has hosted [a download link](https://apkmirror.com/apk/xiaomi-inc/mihome/mihome-5-4-49-release/mi-home-5-4-49-android-apk-download)
  2. Run app and create an account:
    - Choose to allow write to storage. The other permissions requests can be rejected.
    - Choose the China mainland region
    - Sign in using password, then click on sign up link
    - Select China mainland region (which mandates sign up using phone number only)
    - Tested to be working with Google Voice number: click on `+86` to change country code to `+1`, then use your Google voice number
    - Follow the prompts to create your account.
    - Click on the back arrow to return to the login page when done. Click on `Password` option again, then provide your Google voice number and password to sign in.
  3. Upgrade to a recent version of Mi Home
    - verified with version 7.0.704 [arm64-v8a](https://apkmirror.com/apk/xiaomi-inc/mihome/mihome-7-0-704-release/mi-home-7-0-704-2-android-apk-download); beware of the bundle version on apkmirror that needs its own installer
    - see the [findings](#findings) section below on rationale
    - launch the updated app. You should still be signed in.
  4. Proceed to pair with the device
    - Agree to bluetooth and location permission
    - Press the Yeelight S1 once to wake it up, then go to add device in app to discover the device. Follow prompts to add the device to your account.
    - Verify that your device is added to your account, then uninstall Mi Home app.
  5. Install version 5.4.49 of MiHome again.
    - Follow step 2 except that you don't need to sign up this time
    - Sign in and wait for things to sync up. Devices bound to your account should appear.
    - It took about 1 minute in our test for the Yeelight S1 to appear.
  6. Copy out `SmartHome/logs/<date>.log` or `SmartHome/logs/plug_DeviceManager/<date>.log` to your PC.
    - Open with notepad and look for the line `SmartHome ...[DEBUG]... processResult in result=...`
    - Everything after `result=` is a JSON. You should be able to find the token for your device in there.
  7. Go ahead and uninstall Mi Home on your phone.
    - These directories are created by Mi Home and can be deleted too: `.vdevdir`, `mipush`, `SmartHome`, `Xiaomi`, `amap`.


Findings
--------
Xiaomi devices may be region locked. On version 5.4.49, if you set region to US, Yeelight S1 doesn't appear in discovery, but does so when you change the region to China mainland.

Version 5x doesn't seem to support pairing with Yeelight S1. It sees the device in discovery but fails with error `failed to verify (-27)` when pairing.
