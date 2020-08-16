# HTPC

## Install Kodi

Install xpra

sudo apt-get install libx11-dev libxtst-dev libxcomposite-dev \
    libxdamage-dev libxkbfile-dev xauth x11-xkb-utils xserver-xorg-video-dummy \
    python-all-dev python-gobject-2-dev python-gtk2-dev cython \
    libx264-dev libvpx-dev node-uglify yui-compressor

https://xpra.org/src/
  
wget https://xpra.org/src/xpra-4.0.3.tar.xz
tar -Jxf xpra-4.0.3.tar.xz
rm xpra-4.0.3.tar.xz
cd xpra-4.0.3/
python3 ./setup.py build
sudo python3 ./setup.py install

sudo apt install python3-pip
pip3 install --upgrade cython
sudo apt install libgtk-3-dev
(export PKG_CONFIG_PATH=/usr/lib/pkgconfig)
sudo apt-get install python3-cairo-dev




$ sudo apt install xpra
$ xpra --version
xpra v2.4.3-r21350M



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

## Install Steam Link
