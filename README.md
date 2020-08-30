# HTPC

## Configure bluetooth controllers

### Hardware

- [Xbox One S Controller][Xbox Wireless Controller] and/or
- [PS4 Controller]
- The in-built Bluetooth 5.0 of the Raspberry Pi 4

### Features

The Xbox One S Wireless Controller and the PS4 Controller configured according
to this guide are perfectly supported by [Steam Link for Raspberry Pi].

- These controllers automatically reconnect to the Raspberry Pi when they are
  powered off/on.
- The mapping of the buttons is identical to when they are directly connected to
  a computer running Steam.
- The vibration functions of these controllers are fully supported when using
  Steam Link (confirmed when playing to [Cuphead])
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

Enable the bluetooth service to start at boot and check its status:

    sudo systemctl enable bluetooth
    sudo systemctl status bluetooth

### Disable ERTM

The Bluetooth *enhanced retransmission mode (ERTM)* is enabled by default on the
Raspberry Pi and prevents the Xbox One S controller to connect. From the
interface of `bluetoothctl` and when ERTM is enabled, attempting to connect to
the Xbox One S Controller (see next section) should result in the following loop.

    [bluetooth]# connect XX:XX:XX:XX:XX:XX
    Attempting to connect with XX:XX:XX:XX:XX:XX
    [CHG] Device XX:XX:XX:XX:XX:XX Connected: yes
    [CHG] Device XX:XX:XX:XX:XX:XX Connected: no
    [CHG] Device XX:XX:XX:XX:XX:XX Connected: yes
    [CHG] Device XX:XX:XX:XX:XX:XX Connected: no
    ...

ERTM is not needed for our application and can be safely disabled. We can
temporarily disable ERTM until the next reboot with the first command. The second
command confirms that ERTM has been successfully disabled.

    $ sudo bash -c 'echo 1 > /sys/module/bluetooth/parameters/disable_ertm'
    $ cat /sys/module/bluetooth/parameters/disable_ertm
    Y

If `bluetoothctl` is running when we disabled ERTM, we may need to exit and
run it again before attempting to connect to the controller. We can use the
command below to permanently disable ERTM starting from the next reboot.

    sudo bash -c "echo 'options bluetooth disable_ertm=Y' > /etc/modprobe.d/bluetooth.conf"

### Connect the controllers

Enter the following command on the Raspberry Pi to run `bluetoothctl`.

    sudo bluetoothctl

Now that we are in the Bluetooth command-line tool, we need to go ahead and turn
the agent as well as start scanning for other devices. Enter `help` for a list
of all the options available.

    agent on
    scan on

We should start seeing an output similar to the one below. The MAC addresses
featured in this guide have been replaced by random values for security reason.

    [bluetooth]# scan on
    Discovery started
    [CHG] Controller C6:DA:27:5F:11:95 Discovering: yes
    [NEW] Device 4F:C3:E7:40:98:6F 4F:C3:E7:40:98:6F
    [NEW] Device 54:E8:75:3B:CB:F7 BRAVIA 4K UR2

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
  driver, however this does not translates in any noticeable issue.

  Xbox One S controller:

      (EE) libinput bug: Event for missing capability CAP_POINTER on device "Xbox Wireless Controller"

  PS4 controlller:

    <!-- markdownlint-disable MD032 MD034 -->
      (EE) event1  - Wireless Controller Touchpad: kernel bug: Touch jump detected and discarded.
      See https://wayland.freedesktop.org/libinput/doc/1.12.6/touchpad-jumping-cursors.html for details

- We can check the buttons and axis response of the controllers using the
  command-line tool `jstest`. First, we need to identify the `/dev/input` handler
  of our controllers by entering `cat /proc/bus/input/devices`. These handlers
  can not be used to uniquely identify controllers and depends on the order the
  controllers have been connected to the Raspberry Pi. The first controller
  connected should receive the handler `js0`, the second `js1`, etc. If one controller
  is disconnected, its handler will become available to the next controller to
  connect. Once we have identified the handler of the controller that we want to
  test, we can start `jstest`. For example:

        $ jstest --normal /dev/input/js0
        Driver version is 2.1.0.
        Joystick (Xbox Wireless Controller) has 8 axes (X, Y, Z, Rz, Gas, Brake, Hat0X, Hat0Y)
        and 15 buttons (BtnA, BtnB, BtnC, BtnX, BtnY, BtnZ, BtnTL, BtnTR, BtnTL2, BtnTR2, BtnSelect, BtnStart, BtnMode, BtnThumbL, BtnThumbR).
        Testing ... (interrupt to exit)
        Axes:  0:     0  1:     0  2:     0  3:     0  4:-32767  5:-32767  6:     0  7:     0 Buttons:  0:on   1:off  2:off  3:off  4:off  5:off  6:off  7:off  8:off  9:off 10:off 11:off 12:off 13:off 14:off sad

## Connect a wireless keyboard

It is strongly recommended to connect a wireless keyboard to our Raspberry Pi HTPC
so that we can interact with unexpected graphical prompts or whenever using a
controller would not result in the best user experience, for example when
browsing the web using [Steam Link][Steam Link for Raspberry Pi] browser. If you
have installed Raspberry Pi Desktop to interact with GUI program, having as
wireless keyboard is a must-have!

A keyboard that works out of the box is the [Logitech Wireless Touch Keyboard K400].
Simply plug the tiny Logitech Unifying receiver into a USB port and you're ready
to start browsing or typing.

### Change keyboard layout to US from default British layout

The default keyboard layout on Raspberry Pi OS is a British layout. When using
the keyboard K400 with US layout, this configuration makes it difficult to enter
the backslash and vertical bar (pipe) keys. The solution is to change the value
of the variable `XKBLAYOUT` from `gb` to `us` in `/etc/default/keyboard` before
restarting the system.

    $ sudo cat /etc/default/keyboard
    XKBMODEL="pc105"
    XKBLAYOUT="gb"
    XKBVARIANT=""
    XKBOPTIONS=""

    BACKSPACE="guess

## Configure Video

### Enable monitor hotplug

It is possible that the display shows No Signal when is has been turned on after
the Raspberry Pi has boot up.

A solution is to set `hdmi_force_hotplug=1` in `/boot/config.txt`. What this
change effectively does is that the HDMI output mode will be used, even if no
HDMI monitor is detected. To test this solution, turn off the monitor and
restart the Pi with `sudo reboot`. Wait a couple of minutes then turn on your
monitor to check that it receives signal.

A potential problem that occurs when the Raspberry Pi is turned on and the
monitor is off when `hdmi_force_hotplug=1` is that the Pi is unable to identify
the resolutions supported by the monitor. In this case, the Pi will settle for a
low resolution like 640x480. A solution to this problem is to specify the
resolution that the Pi should use at startup (see below).

### Specify the default screen resolution

The easiest solution to set the default screen resolution that the Raspberry Pi
must use among the resolutions supported by the TV is to use `sudo raspi-config`
\> `Advanced Options` > `Resolution`. The resolutions listed depends on whether
the TV was connected at startup. For example, the resolution 4kp60Hz will only
be listed if the TV supports it and was turned up at startup.

> Raspberry 4B: The Raspberry Pi HDMI port adjacent to the USB-C power input
(labelled HDMI0) is the only port that supports 4K with a 60Hz refresh rate. If
both ports are used, their resolution is limited to 1080p @ 30Hz.

Alternatively, use the following commands to identify the acceptedd values for
`hdmi_group` and `hdmi_mode` before specifying them in `/boot/config.txt`

To show the current resolution:

    $ /opt/vc/bin/tvservice -s
    state 0x6 [DVI CUSTOM RGB full 16:9], 1920x1080 @ 60.00Hz, progressive

To show the resolution accepted (`CEAs`: TVs, `DMTs`: monitors):

    $ /opt/vc/bin/tvservice -m CEAs
    Group CEA has 4 modes:
           mode 4: 1280x720 @ 60Hz 16:9, clock:74MHz progressive
    (prefer) mode 16: 1920x1080 @ 60Hz 16:9, clock:148MHz progressive
            mode 95: 3840x2160 @ 30Hz 16:9, clock:297MHz progressive
            mode 97: 3840x2160 @ 60Hz 16:9, clock:594MHz progressive

See [Video options in config.txt] for more information on the Raspberry Pi
video options.

### Remove the black border from the display

If a black border of visible pixel surrounds the video content, the solution could
be to turn off the overscan. Try setting the value `disable_overscan=1` in
`/boot/config.txt`. If this does not solve the issues, try setting the following
values manually:

    overscan_left=16
    overscan_right=16
    overscan_top=16
    overscan_bottom=16

### Set gpu_mem

The Raspberry Pi allows us to specify how much memory, in megabytes, to reserve
for the exclusive use of the GPU on Raspberry Pi 1-3. The remaining memory iss
allocated to the ARM CPU. On the Raspberry Pi 4 the 3D component of the GPU has
its own memory management unit (MMU), and does not use memory from the gpu_mem
allocation. Instead memory is allocated dynamically within Linux. This may allow
a smaller value to be specified for gpu_mem on the Pi 4, compared to previous
models.

On Raspberry Pi 1 to 3, we would set the value of `gpu_mem` in `/boot/config.txt`
up to the value listed on the page below that depends on the total RAM available
on the Pi. According to this page, the Raspberry Pi 4 does not rely on `gpu_mem`
so there is no need to set it.

See [Memory options in config.txt] for more information on the Raspberry Pi
CPU and GPU options.

### Monitor GPU usage

TODO

## Configure Audio

<!-- markdownlint-disable MD024 -->
### Hardware

- Jack audio speaker
- HDTV with HDMI ports
- [Micro-HDMI to HDMI TV Adapter Cable][uhdmi_hdmi_cable]
  - Recommended features: 4K Video at 60 Hz, Audio Return Channel (ARC)

### Test audio output devices

First we need to check that our audio hardware is functional before moving on with
the software configuration. For this test, we use the video player [Omxplayer]
specifically made for the Raspberry Pi's GPU from the Kodi project. Omxplayer
player is installed by default on Raspbian OS Lite.

### Test jack audio output

Connect your speaker or headset to the jack port of the Raspberry Pi. Run the
following commands to download and play an mp3 sample file with the audio output
directed to the jack port (`-o local`).

    curl -O https://raw.githubusercontent.com/tschaffter/raspberry-pi-htpc/master/audio/example.mp3
    omxplayer -o local example.mp3

### Test HDMI audio ouput

Connect your HDTV to the Raspberry Pi using a recent micro-HDMI to HDMI cable.
Over the years, many improvements have been brought by different versions
of the HDMI protocol and hardware to continuously improve audio support. Here we
connect the cable to the Raspberry Pi HDMI port adjacent to the USB-C power input
(labelled HDMI0) because only this port supports 4K with a 60Hz refresh rate.

Download the mp3 sample file if not done previously, and run `omxplayer` with
the argument `-o hdmi` to direct the audio to the TV speakers.

    curl -O https://raw.githubusercontent.com/tschaffter/raspberry-pi-htpc/master/audio/example.mp3
    omxplayer -o hdmi example.mp3

TODO: if audio does not go through: https://www.raspberrypi.org/documentation/configuration/audio-config.md
https://www.raspberrypi.org/documentation/usage/audio/

### Install PulseAudio

PulseAudio is a sound system for Linux – this means that it works as a proxy
between your audio hardware and programs that want to play sounds.

## Overclocking

### Increase CPU frequency

Checking the default speed of CPU:

    $ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
    600000

Chances are you’ll receive 600000 back from the serial terminal, which would indicate your Pi 4 CPU base speed is 600MHz

watch -n 1 vcgencmd measure_clock arm

> NOTE: Do note that from testing, some Raspberry Pi 4 boards have failed to boot at this speed or slowed down due to overheating/undervoltage. It’s unlikely for you to maintain your Pi 4 board at this max speed in the long run, hence we recommend you to settle for arm_freq=2000 or stop at step 5.

See yellow settings in:
https://miguelangelantolin.com/raspberry-pi-4b-overclock/

sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run

https://retropie.org.uk/forum/topic/27430/howto-optimized-boot-config-txt


From https://www.raspberrypi.org/forums/viewtopic.php?t=283911&p=1718846:
#over_voltage=2
#arm_freq=1800

over_voltage=5
arm_freq=2000

#over_voltage=6
#arm_freq=2100

$ vcgencmd measure_volts core
volt=0.8350V

## Steam Link

### Configure the Host

The term "host" refers to the computer that is running [Steam] and streaming the
video and audio to the Raspberry Pi. In this guide, the host is a computer running
Windows 10, however any OS that supports [Steam] should work.

For optimal performance, we connect the host to the router/switch using a Cat 6
ethernet cable. Cat-6 cables have a data transfer rate of 1 gigabits per second
(Gb/s). This is more than enough to stream GAME_NAME at RESOLUTION @ 60Hz.

We just have to start Steam on the host and we are good to go.

### Install Steam Link on the Raspberry Pi

We are going to install [Steam Link] on the Raspberry Pi, which will later
connect to the computer running [Steam] (host).

The easiest way to install Steam Link on Raspberry OS is to use the package
manager. However, keep reading to learn about alternative options.

Using Raspberry OS package manager:

    $ apt-cache policy steamlink
    steamlink:
    Installed: (none)
    Candidate: 1.0.7
    Version table:
        1.0.7 500
            500 http://archive.raspberrypi.org/debian buster/main armhf Packages
    $ sudo apt install -y steamlink

As of August 29, 2020, the latest version of Steam Link on Raspberry Pi is
`1.1.64.162` as advertised on the top of a pinned post on the [official Steam
Link discussion board](https://steamcommunity.com/app/353380/discussions/6/).
The same information can be found programmatically using the command below

    $ curl -Ls http://media.steampowered.com/steamlink/rpi/public_build.txt
    http://media.steampowered.com/steamlink/rpi/steamlink-rpi3-1.1.64.162.tar.gz

Steam Link uses the above link to check if a more recent version than the one
installed is available. If yes, the latest version is downloaded and installed
in `~/.local/share/SteamLink/` before being executed. Thus, the initial version
of Steam Link installed does not matter much. On Raspbian OS, the easiest option
is to install Steam Link using the package manager. On Linux distrbutions that
do not provide it as a package, the above command can be used to download and
install the latest version of Steam Link.

> Note: `public_build.txt` can be replaced by `beta_build.txt` to access the
beta version. Alternatively, the beta version of Steam Link can be activated
after launching its stable (publlic) version.

On Debian-based distributions, there is a more recent package available
than the one provided by (Raspbian OS) package manager. This link was found by
bumping up the version of a similar link found online for `1.0.7`.

    $ curl -Os http://media.steampowered.com/steamlink/rpi/steamlink_1.0.8_armhf.deb
    $ sudo apt install steamlink_1.0.8_armhf.deb
    $ apt-cache policy steamlink
    steamlink:
    Installed: 1.0.8
    Candidate: 1.0.8
    Version table:
    *** 1.0.8 100
            100 /var/lib/dpkg/status
        1.0.7 500
            500 http://archive.raspberrypi.org/debian buster/main armhf Packages

Even though the package manager shows that the version installed is `1.0.8`,
Steam Link will actually download and run the latest version available
(currently `1.1.64.162`).

### Start Steam Link

Steam Link requires an X server to render the video on the TV. Attempting to
start Steam Link without an X server would generate the following message.

    $ steamlink
    * failed to add service - already in use?

The default X server adopted by most Linux distributions, including Raspberry OS
Desktop, is X.Org Server (X11). We go ahead and install the command `startx`
from the package `xinit` to launch X11 sessions, as well as the standard terminal
emulator for the X Window System, `xterm`.

    sudo apt-get install --no-install-recommends xserver-xorg xinit xterm

The command `startx` should not be run with `sudo` for safety reasons, thus
preventing permission problems that may break the GUI. When using a keyboard
connected to the Raspberry Pi to interact with the default "physical" terminal
(`tty1`), we can start an X session as a non-root user with the command `startx`.

The X session should look like a black screen with an instance of the terminal
`xterm` at the top-right corner of the screen. We can switch between the
terminals `tty1` to `tty7` by pressing `Ctrl+Alt+F1-F7`. We can then go back to
the X session attached to `tty1` with the shortcut `Ctrl+Alt+F1`.

From a terminal, we can list the processes related to Xorg:

    $ ps aux | grep Xorg
    tschaff+  1027  0.1  1.1 122432 43984 tty1     Sl   18:44   0:00 /usr/lib/xorg/Xorg -nolisten tcp :0 vt1 -keeptty -auth /tmp/serverauth.tsUxEiGH6h
    tschaff+  1104  0.0  0.0   7348   548 pts/1    S+   18:52   0:00 grep --color=auto Xorg

The first line shows that an X session (`/usr/lib/xorg/Xorg`) is attached to the
terminal `tty1`. The second line refers to the SSH terminal (`pts/1`) that wes
are using to run the above command.

The X session that we have started can be stopped with `kill <pid>` where `<pid>`
is the process ID of the session listed by `ps aux | grep Xorg`. Here, we would
run `kill 1027` to stop the X session.

By default a non-root user is not authorized to start an X session from an SSH
terminal. Adding the non-root user to the group `tty` and `video` solves some of
the error messages but not all of them. The easiest solution found is to install
enable any user to start an X session in the file `/etc/X11/Xwrapper.config`
provided by the package `xserver-xorg-legacy`.

    $ sudo apt install -y xserver-xorg-legacy
    $ cat /etc/X11/Xwrapper.config
    allowed_users=anybody

We can now run `startx` as a non-root user from an SSH terminal.





    sudo usermod --append --groups tty $(whoami)

sudo apt install xserver-xorg-legacy


![steamlink_controllers_list](pictures/steamlink_controllers_list.png)

<!--   


Audio lag in HDMI is fairly common, but the causes are typically either video processing delay (which actually causes the video to lag, not the audio), or a sync problem in the case of video broadcasts.



On your host computer, find the audio playback settings and change your playback sample rate.

44100Hz and 48000Hz playback sample rates usually work well for streaming. Some users find that switching from one to the other fixes distortion or crackling. 

Yes. Seems to happen if there's ANY hiccups in FPS.

check power, could also be a ground issue

And...turning the display setting to anything other than 1080p fixed it. It must be the TV's deinterlacing process interfering with the Steam Link's.

Inhome streaming only properly supports 48KHz audio and MacOS by default uses 44.1KHz




Decreasing sound using alsamixer: crackling in the menu still present

Here is one way you could do it. (in /etc/asound.conf or ~/.asoundrc)

aplay example.mp3 is distorded!
omxplayer -o hdmi example.mp3 is OK

For wav you can use aplay. For mp3 you can use mpg123

$ aplay example.wav
Playing WAVE 'example.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Stereo

Most of the audio on TV is fine but crackling clearly due to TV broken speakers



=> Try bluetooth headphones


Update host GPU driver
Music on host pause when Steam Link start, then resume when turning Steam Link
    off.







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
[Omxplayer]: https://www.raspberrypi.org/documentation/raspbian/applications/omxplayer.md
[uhdmi_hdmi_cable]: https://www.amazon.com/gp/product/B014I8U6N0
[Memory options in config.txt]: https://www.raspberrypi.org/documentation/configuration/config-txt/memory.md
[Kodi]: https://kodi.tv/
[Logitech Wireless Touch Keyboard K400]: https://www.logitech.com/en-us/product/wireless-touch-keyboard-k400r
[Video options in config.txt]: https://www.raspberrypi.org/documentation/configuration/config-txt/video.md
[Steam]: https://store.steampowered.com/about/
[Steam Link now available on Raspberry Pi]: https://steamcommunity.com/app/353380/discussions/6/2806204039992195182/
[X server]: https://en.wikipedia.org/wiki/X_server