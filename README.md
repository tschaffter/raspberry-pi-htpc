# HTPC

## Configure bluetooth controllers

### Hardware

- [Xbox One S Controller][Xbox Wireless Controller] and/or
- [PS4 Controller]
- The in-built Bluetooth 5.0 of the Raspberry Pi 4

### Features

The Xbox One S Wireless Controller and the PS4 Controller configured according
to this guide are perfectly support by [Steam Link for Raspberry Pi].

- These controllers automatically reconnect to the Raspberry Pi when they are
  powered off/on.
- The mapping of the buttons is identical to when they are directly connected to
  a computer running Steam.
- The vibration functions of these controllers are fully supported when using
  Steam Link (confirmed when playing [Cuphead])
- These controllers can be be simulatenously connected (confirmed with one Xbox 
  One S Wireless Controller and two PS4 controllers)

### Install bluetoothctl

`bluetoothctl` is the main command for configuring Bluetooth devices on Linux.
Contrary to what the name's structure might lead you to expect, bluetoothctl is
not part of systemd, but rather a simple set of options for setting up Bluetooth
devices.

If you are running the latest Raspberry Pi OS, then all the software has already
been installed. If not, then you can simply type the following to install the
Bluetooth module.

    sudo apt-get install pi-bluetooth

### Connect the controllers

Enter the following command on the Raspberry Pi to run `bluetoothctl`.

    bluetoothctl

Now that we are in the Bluetooth command-line tool, we need to go ahead and turn
the agent as well as start scanning for other devices. Enter `help` for a list
of all the options available.

    agent on
    scan on

We should start seeing an output similar to this one:

    [bluetooth]# scan on
    Discovery started
    [CHG] Controller DC:A6:32:05:7F:06 Discovering: yes
    [NEW] Device 51:B8:16:6A:6F:C6 51-B8-16-6A-6F-C6
    [NEW] Device 40:23:43:3F:4E:58 BRAVIA 4K UR2

The "Controller" prefix refers to the Raspberry Pi itself. "Devices" correspond
to the devices detected by the Raspberry Pi. From now on, we save the term
"controller" for the gaming controllers.

We can now turn on the pairing mode of the controller.

- [Xbox One S Controller][Xbox Wireless Controller]: Turn on the controller by
  pressing the Home button for less than one second. The light of the Home button
  should start to blink slowly, indicating that the controller is searching for
  a connection signal. Now press the Connect button on the top side of the
  controller to enable the pairing mode, which will results in making the Home
  button to blink faster.
- [PS4 Controller]: Unplug the power from your PS4 or turning on the controller
  will turn it on. Press and hold the PS and Share buttons on the controller at
  the same time. The light bar on the back of the controller will start flashing
  once pairing mode is active.

Now that the controller is discoverable, wait for the scan output of `bluetoothctl`
to show the controller. At the time of writing this guide, the Xbox One S Controller
should appear as "Xbox Wireless Controller" and the PS4 controller as "Wireless
Controller". Identify the MAC address of the controller (XX:XX:XX:XX:XX:XX) and
use it in the following commands:

    pair XX:XX:XX:XX:XX:XX
    trust XX:XX:XX:XX:XX:XX
    connect XX:XX:XX:XX:XX:XX

If the output of `connect XX:XX:XX:XX:XX:XX` is `yes`, the controller should now
be connected to the Raspberry Pi. The Home button of the Xbox One Wireless
controller should turn solid white and the main light of the PS4 controller should
turn solid blue for the first PS4 controller connect, red for the second, etc.

### Notes

- [xpadneo] is an Advanced Linux Driver for Xbox One Wireless Gamepad. This driver
  advertises the support of all Force Feedback/Rumble effects, Trigger Force
  Feedback and other features. I don't know if these features are exclusive to
  this driver or if they are now also part of the default driver used in this
  guide. I have successfully installed this driver but have uninstalled it after
  observing that Steam Link sometimes double-detect the controller, whcih results
  in every button pressed behaving as pressed twice. [xpadneo] also changes the
  the default mapping of the controller buttons. When this driver is used, the
  Xbox One S Controller vibrates upon connection.
- The command `steamlink` logs the folowing warnings/errors when using the default
  driver, however this does not result in any noticeable issue.

  Xbox One S controller:

      (EE) libinput bug: Event for missing capability CAP_POINTER on device "Xbox Wireless Controller"

  PS4 controlller:

      <!-- markdownlint-disable MD032 MD034 -->
      (EE) event1  - Wireless Controller Touchpad: kernel bug: Touch jump detected and discarded.
See https://wayland.freedesktop.org/libinput/doc/1.12.6/touchpad-jumping-cursors.html for details







<!--   










## Install Kodi



https://dustinpfister.github.io/2020/03/27/linux-raspbian-lite-xserver-xorg/

sudo apt-get install -y xserver-xorg
sudo apt-get install -y xinit

Edit (or create) the file /etc/X11/Xwrapper.config with the following content:

allowed_users=anybody
needs_root_rights=yes

black box: window manager
lightdm: display manager

codafog/kodi-rpi: not updated in 3 years + error


## Install RetroPie

## Install Steam Link -->


<!-- Definitions -->

[Xbox Wireless Controller]: https://www.xbox.com/en-US/accessories/controllers/xbox-wireless-controller
[PS4 Controller]: https://www.playstation.com/en-us/explore/accessories/gaming-controllers/
[Steam Link for Raspberry Pi]: https://support.steampowered.com/kb_article.php?ref=6153-IFGH-6589
[Cuphead]: https://store.steampowered.com/app/268910/Cuphead/
[xpadneo]: https://github.com/atar-axis/xpadneo