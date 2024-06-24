FreeSWITCH Installation on Ubuntu 22.04 LTS

FreeSWITCH Installation Guide for Ubuntu 22.04 LTS
This guide outlines the steps to install FreeSWITCH on Ubuntu 22.04 LTS. FreeSWITCH is a powerful open-source telephony platform that can be used to build various communication applications.

Installation Steps

### Step 1: Install FreeSWITCH Dependencies

apt install --yes build-essential pkg-config uuid-dev zlib1g-dev libjpeg-dev libsqlite3-dev libcurl4-openssl-dev libpcre3-dev libspeexdsp-dev libldns-dev libedit-dev libtiff5-dev yasm libopus-dev libsndfile1-dev unzip libavformat-dev libswscale-dev liblua5.2-dev liblua5.2-0 cmake libpq-dev unixodbc-dev autoconf automake ntpdate libxml2-dev libpq-dev libpq5 sngrep lua5.2 lua5.2-doc libreadline-dev

### Step 2: Configure Timezone

To configure the timezone, run:
dpkg-reconfigure tzdata

Edit the crontab:
crontab -e

Add the following line to run the ntpdate command every 30 minutes:
*/30 * * * * ntpdate #Enter your IP Here

### Step 3: Install libspandsp3

Clone the spandsp repository:
git clone https://github.com/freeswitch/spandsp.git

Navigate to the cloned repository:
cd spandsp/

Run the bootstrap, configure, make, and install commands:
./bootstrap.sh && ./configure && make && make install

### Step 4: Install sofia-sip

Download the sofia-sip repository:
cd /usr/local/src/
wget "https://github.com/freeswitch/sofia-sip/archive/master.tar.gz" -O sofia-sip.tar.gz

Extract the downloaded tarball:
tar -xvf sofia-sip.tar.gz

Navigate to the extracted repository:
cd sofia-sip-master

Run the bootstrap, configure, make, and install commands:
./bootstrap.sh && ./configure && make && make install

### Step 5: Download FreeSWITCH

Download the FreeSWITCH source:
cd /usr/local/src/
wget https://files.freeswitch.org/releases/freeswitch/freeswitch-1.10.10.-release.tar.gz

Extract the downloaded tarball:
tar -zxvf freeswitch-1.10.10.-release.tar.gz

### Step 6: Install Lua Module

Copy Lua headers:
cp /usr/include/lua5.2/*.h  /usr/local/src/freeswitch-1.10.10.-release/src/mod/languages/mod_lua/

Create a symbolic link for Lua:
sudo ln -s /usr/lib/x86_64-linux-gnu/liblua5.2.so /usr/lib/x86_64-linux-gnu/liblua.so

### Step 7: FreeSWITCH Installation

Navigate to the FreeSWITCH source directory:
cd freeswitch-1.10.10.-release

Open modules.conf in your favorite editor and remove or comment mod_signalwire and mod_verto.

Configure FreeSWITCH:
./configure --enable-core-odbc-support --enable-core-pgsql-support

Compile FreeSWITCH:
make

Install FreeSWITCH:
make install

Install CD Sounds and Music:
make cd-sounds-install
make cd-moh-install

### Step 8: Configure FreeSWITCH

Create softlinks and paths:
ln -s /usr/local/freeswitch/conf /etc/freeswitch
ln -s /usr/local/freeswitch/bin/fs_cli /usr/bin/fs_cli
ln -s /usr/local/freeswitch/bin/freeswitch /usr/sbin/freeswitch

### Step 9: Add Non-Root User for FreeSWITCH

Add the freeswitch group:
groupadd freeswitch

Add the freeswitch user:
adduser --quiet --system --home /usr/local/freeswitch --gecos 'FreeSWITCH open source softswitch' --ingroup freeswitch freeswitch --disabled-password

Set permissions for the freeswitch user:
chown -R freeswitch:freeswitch /usr/local/freeswitch/
chmod -R ug=rwX,o= /usr/local/freeswitch/
chmod -R u=rwx,g=rx /usr/local/freeswitch/bin/*

### Step 10: Set Up Systemd Service

Open /etc/systemd/system/freeswitch.service in your favorite editor and paste the following contents:

[Unit]
Description=freeswitch
Wants=network-online.target
Requires=network.target local-fs.target
After=network.target network-online.target local-fs.target

[Service]
; service
Type=forking
PIDFile=/usr/local/freeswitch/run/freeswitch.pid
Environment="DAEMON_OPTS=-nonat"
Environment="USER=freeswitch"
Environment="GROUP=freeswitch"
EnvironmentFile=-/etc/default/freeswitch
ExecStartPre=/bin/chown -R ${USER}:${GROUP} /usr/local/freeswitch
ExecStart=/usr/local/freeswitch/bin/freeswitch -u ${USER} -g ${GROUP} -ncwait ${DAEMON_OPTS}
TimeoutSec=45s
Restart=always

[Install]
WantedBy=multi-user.target


Reload the systemd daemon:
systemctl daemon-reload

Enable the FreeSWITCH service to start on boot:
systemctl enable freeswitch.service

Start the FreeSWITCH daemon:
systemctl start freeswitch.service

Check if the daemon is loaded successfully:
systemctl status freeswitch.service

If the daemon is not started, run:
ldconfig
