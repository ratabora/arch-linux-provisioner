[![Stories in Ready](https://badge.waffle.io/ratabora/arch-linux-provisioner.png?label=ready&title=Ready)](https://waffle.io/ratabora/arch-linux-provisioner?utm_source=badge)
# arch-linux-provisioner
Provision an Arch Linux installation. Heavily inspired by [pigmonkey/spark](https://github.com/pigmonkey/spark)

## How to run

```
# ansible-playbook -i localhost playbook.yml
```

## Razer Stealth
### Fix lid close state

https://wiki.archlinux.org/index.php/razer#Suspend_Loop
https://bugzilla.kernel.org/show_bug.cgi?id=187271

1. disable lid events for systemd:
Add `HandleLidSwitch=ignore` to /etc/systemd/logind.conf

2. add a lid close event handler to acpid (/etc/acpi/events/lid):
event=button[ /]lid LID close
action=/etc/acpi/lid.sh

3. create /etc/acpi/lid.sh which after prepping for suspend calls:
echo -n "mem" > /sys/power/state

4. add `button.lid_init_state=open` to the kernel params and reboot.

https://wiki.archlinux.org/index.php/Kernel_parameters#systemd-boot

Note: If you have not set a value for menu timeout, you will need to hold Space while booting for the systemd-boot menu to appear.
To make the change persistent after reboot, edit /boot/loader/entries/arch.conf (assuming you set up your EFI System Partition) and add them to the options line:
options root=/dev/sda2 quiet splash
For more information on configuring systemd-boot, see the systemd-boot article

### Chroma
  548  yaourt -S razer-driver-meta
  551  pip install dbus-python
  552  yaourt -S python-notify2
  553  yaourt -S razer-daemon
  555  sudo gpasswd -a ratabora plugdev
  557  yaourt -S razer-driver-meta
  559  yaourt -S polychromatic
 
