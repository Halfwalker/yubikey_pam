## Yubikey authentication in Linux with Gnome

This will set up a Gnome desktop to use a Yubikey to authenticate for login and sudo access.  Also, removing the Yubikey will automatically lock the desktop.

### Requirements

`finger`, `gnome-screensaver` and `libpam-u2f` must be installed.  `finger` is used to find the current user X11 display number.  `gnome-screensaver` is used to actually lock the desktop. `libpam-u2f` provides PAM with functionality to use the Yubikey.

```
sudo apt install finger gnome-screensaver libpam-u2f
```

### Create a keys storage location

You need a generic file for holding keys.  Then add your Yubikey key to that file.

```
pamu2fcfg | sudo tee /etc/u2f_keys
```

### Removal of Yubikey to lock workstation

Check the device ID of your Yubikey
```
$ lsusb | grep -i yubi

Bus 001 Device 018: ID 1050:0407 Yubico.com
```

Ensure the *ID_VENDOR_ID* and *ID_MODEL_ID* entries in the `65-yubikey.lock.rules` file are correct.
```
# Looking for Yubikey removal
# Bus 001 Device 011: ID 1050:0407 Yubico.com
ACTION=="remove", ENV{ID_VENDOR_ID}=="1050", ENV{ID_MODEL_ID}=="0407", RUN+="/usr/local/bin/gnome-screensaver-lock"
```

* Install the `gnome-screensaver-lock` script into _/usr/local/bin_.
* Install the `65-yubikey.lock.rules` udev rule into _/etc/udev/rules.d_.

Activate the new udev rule
```
sudo udevadm trigger
```

Removing the Yubikey should immediately lock the workstation.

### Using Yubikey auth for login and sudo

Edit _/etc/pam.d/common-auth_ to add the following to the bottom of the file :
```
auth  required  pam_u2f.so  nouserok authfile=/etc/u2f_keys cue
```

Now when logging in or performing a sudo cmd, you will be prompted to touch the Yubikey.

