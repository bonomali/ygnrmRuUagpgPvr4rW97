# qBittorrent-nox (Debian 9 64bits)
For FeralHosting Seedbox and UltraSeedBox  
Bulit on Oct.14.2018, with Qt 5.7.1, libboost 1.62, libtorrent-rasterbar RC_1_0 (1.0.11)  

## Building Notes


### Building dependency

```
cd && mkdir -p /root/0.qb/{deb,lib,source} && \
apt-get install -y build-essential pkg-config autoconf automake libtool git checkinstall libboost-dev libboost-system-dev libboost-chrono-dev libboost-random-dev libssl-dev geoip-database libgeoip-dev libboost-python-dev libgl1-mesa-dev qtbase5-dev qttools5-dev-tools libqt5svg5-dev zlib1g-dev
```


### Build libtorrent-rasterbar

```
branch=RC_1_0 ; version=1.0.11
git clone --depth=1 -b $branch https://github.com/arvidn/libtorrent libtorrent-$version
cd libtorrent-$version

sed -i "s/+ target_specific(),/+ target_specific() + ['-std=c++11'],/" bindings/python/setup.py && \
./autotool.sh && \
./configure --enable-python-binding --with-libiconv --disable-debug --enable-encryption --with-libgeoip=system CXXFLAGS=-std=c++11 && \
mkdir -p doc-pak
cat >description-pak<<EOF
an efficient feature complete C++ bittorrent implementation
EOF

make -j$(nproc)
checkinstall -y --pkgname=libtorrent-rasterbar --pkggroup libtorrent --pkgversion $version
mv -f libtorrent-rasterbar*deb /root/0.qb/deb
cd ; tar zcvpf /root/0.qb/source/libtorrent-1.0.11.tar.gz  /root/libtorrent-1.0.11
```


### Build qBittorrent

```
function _installqb() {
echo -e "\n\n\n$(tput setaf 7)$(tput setab 1)------  qBittorrent $QBVERSION  ------$(tput sgr0)\n\n\n"
cd ; git clone --depth=1 -b release-$QBVERSION https://github.com/qbittorrent/qBittorrent qBittorrent-$QBVERSION ; cd qBittorrent-$QBVERSION

mkdir -p doc-pak
cat >description-pak<<EOF
qBittorrent BitTorrent client headless (qbittorrent-nox)
EOF

./configure --prefix=/usr --disable-gui
make -j$(nproc)
checkinstall -y --pkgname=qbittorrent-nox --pkgversion=$QBVERSION --pkggroup qbittorrent
qbtnox_ver=`qbittorrent-nox --version | awk '{print $2}' | sed "s/v//"`

mv -f qbittorrent*deb /root/0.qb/deb
cp -f $(command -v qbittorrent-nox) /root/0.qb/qbittorrent-nox.$qbtnox_ver
tar zcvpf /root/0.qb/source/qBittorrent-$QBVERSION.tar.gz /root/qBittorrent-$QBVERSION
echo -e "\n\n\n$(tput setaf 7)$(tput setab 1)------ DONE  ------$(tput sgr0)\n\n\n" ; }

qblist="3.3.11  3.3.14  3.3.16  4.0.4  4.1.1  4.1.2  4.1.3"
time { for QBVERSION in $qblist ; do time _installqb ; done ; }
```

### Copy files

```
for ldfiles in "$(ldd $(command -v qbittorrent-nox) | awk '{print $3}')" ; do
cp -f $ldfiles /root/0.qb/lib
done
```

### END

Use SFTP/FTP to download files.



