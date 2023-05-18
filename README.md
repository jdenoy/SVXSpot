# SVXSpot

### Audio and Radio module installation script ###
```console
sudo bash install-radiomodule.sh
```

### SVXLink installation script ###
```console
sudo bash install-svxlink.sh
```

### SVXLink Hotspot in action ###
https://github.com/Guru-RF/SVXSpot/assets/1251767/50dd4366-8439-4067-83b5-5866d0adca77



###Manual install (for now)

apt -y update
apt -y ugprade

# Install the SA818 control software
apt -y install python3-pip git python3-pyaudio python3-scipy
pip3 install sa818

# Install the audio interface driver
# test audio files https://www2.cs.uic.edu/~i101/SoundFiles/
git clone https://github.com/waveshare/WM8960-Audio-HAT
cd WM8960-Audio-HAT/
./install.sh
cd ..

# Disable hdmi-audio and enable serial uart
perl -i -pe 's/dtoverlay=vc4-kms-v3d/dtoverlay=vc4-kms-v3d,noaudio/g' /boot/config.txt
perl -i -pe 's/console=serial0.115200//g'  /boot/cmdline.txt
grep -q 'enable_uart=1' /boot/config.txt || echo "enable_uart=1" >> /boot/config.txt
systemctl disable serial-getty@ttyS0.service

reboot

# edit hotspot (set's frequency and other settings to the Radio Module)
cp hotspot /usr/sbin
chmod a+x /usr/sbin/hotspot

# Only need to run once the module has eeprom
/usr/sbin/hotspot

cp svxspot_volume /usr/sbin/svxspot_volume
chmod a+x /usr/sbin/svxspot_volume

reboot

apt install build-essential g++ make cmake libsigc++-2.0-dev php8.0 libgsm1-dev libudev-dev libpopt-dev tcl-dev libgpiod-dev gpiod libgcrypt20-dev libspeex-dev libasound2-dev alsa-utils libjsoncpp-dev libopus-dev rtl-sdr libcurl4-openssl-dev libogg-dev librtlsdr-dev groff doxygen graphviz python3-serial toilet -y

groupadd svxlink
useradd -g svxlink -d /etc/svxlink svxlink
usermod -aG audio,nogroup,svxlink,plugdev svxlink
usermod -aG gpio svxlink

git clone https://github.com/sm0svx/svxlink.git
mkdir svxlink/src/build

cd svxlink/src/build/

cmake -DUSE_QT=OFF -DCMAKE_INSTALL_PREFIX=/usr -DSYSCONF_INSTALL_DIR=/etc \ -DLOCAL_STATE_DIR=/var -DWITH_SYSTEMD=ON  ..
make
make doc
make install
	
cd /usr/share/svxlink/events.d
ln -s . local

ldconfig

cd /usr/share/svxlink/sounds
wget https://github.com/sm0svx/svxlink-sounds-en_US-heather/releases/download/19.09/svxlink-sounds-en_US-heather-16k-19.09.tar.bz2
tar -xvjf svxlink-sounds-en_US-heather-16k-19.09.tar.bz2
rm *.tar.bz2
ln -s en_US-heather-16k/ en_US

cp svxlink_rotate /usr/sbin
chmod a+x /usr/sbin/svxlink_rotate

ln -s /usr/sbin/svxlink_rotate /etc/cron.daily/svxlink_rotate

cp svxlink.service /lib/systemd/system/svxlink.service

cat gpio.conf > /etc/svxlink/gpio.conf
cat svxlink.conf > /etc/svxlink/svxlink.conf
cat svxlink_gpio_up > /usr/sbin/svxlink_gpio_up

systemctl enable svxlink_gpio_setup
systemctl enable svxlink

systemctl start svxlink_gpio_setup.service
systemctl start svxlink.service







