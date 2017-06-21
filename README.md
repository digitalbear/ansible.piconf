# WiFi for Raspberry Pi

Simple playbook for configuring WiFi for first-time set-up of a new Raspberry Pi device on my local network

This stores the wifi network details in a file encrypted using ansible-vault.  To encrypt or decrypt:
```sh
ansible-vault encrypt secrets.yml --vault-password-file ~/vault_pass.txt
ansible-vault encrypt secrets.yml --vault-password-file ~/vault_pass.txt
```

To retrieve the wifi details from secrets.yml while running the playbook:
```sh
ansible-playbook -i hosts wifi.yml --ask-pass --vault-password-file ~/vault_pass.txt
```
We need to include *--ask-pass* if we have not yet configured an ssh key.  This will prompt us to enter in the ssh password for connecting to the remote device.

### Prerequisites
SSH is no longer enabled by default on your Raspberry Pi.  In order to enable *before* you start up your Pi, while the SD card is still attached to your Macbook (or whatever) after flashing the drive, add an empty file to the boot partition:
```sh
cd /Volumes/boot/
touch ssh
```
