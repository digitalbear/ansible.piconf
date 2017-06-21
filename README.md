# Initial Config for Raspberry Pi

Simple playbook for configuring ssh keys and WiFi for first-time set-up of a new Raspberry Pi device

## While flashing the SD card
### Enable SSH
SSH is no longer enabled by default on your Raspberry Pi.  In order to enable *before* you start up your Pi, while the SD card is still attached to your Macbook (or whatever) after flashing the drive, add an empty file to the boot partition (on your Macbook, not your Pi):
```sh
cd /Volumes/boot/
touch ssh
```

## Start-up the Pi
### Add SSH Key
In order to add ssh key run the ssh role
```sh
ansible-playbook -i hosts ssh.yml --ask-pass
```
We need to include *--ask-pass* since we have not yet configured an ssh key and will need a password to logon.  This will prompt us to enter in the ssh password for connecting to the remote device.  It will copy the default public key from the home directory onto the remote device.  Assuming you have one and there are no errors then you should be able to run the above command again this time without *--ask-pass*

### Configure Wifi
This stores the wifi network details in a file encrypted using ansible-vault.  To encrypt or decrypt:
```sh
ansible-vault encrypt secrets.yml --vault-password-file ~/vault_pass.txt
ansible-vault encrypt secrets.yml --vault-password-file ~/vault_pass.txt
```

To retrieve the wifi details from secrets.yml while running the playbook:
```sh
ansible-playbook -i hosts wifi.yml --vault-password-file ~/vault_pass.txt
```

