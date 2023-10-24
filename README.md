# Build instructions for Asterisk on macOS (Apple Silicon)

Install required homebrew packages. This is my list:
- jansson
- libpq
- lua
- openssl@3
- pkg-config
- portaudio
- postgresql
- sqlite
- srtp
- unixodbc

### Create destination directory, I will use /opt/sangoma:
```bash
sudo install -o `id -un` -g admin -d /opt/sangoma
```

### Clone repo:
```bash
git clone https://github.com/alqemyst/asterisk-macos.git
```

### Downloading and patching pjproject:
```bash
wget https://github.com/pjsip/pjproject/archive/refs/tags/2.13.1.tar.gz
tar xzf 2.13.1.tar.gz && rm 2.13.1.tar.gz

patch -p2 --forward --directory=pjproject-2.13.1 <pjproject.patch
```

### Building pjproject:
```bash
cd pjproject-2.13.1

export PKG_CONFIG_PATH="/opt/homebrew/lib/pkgconfig"
export CFLAGS="-I/opt/homebrew/include -O2 -DNDEBUG"
export LDFLAGS="-L/opt/homebrew/lib"

./configure --prefix=/opt/sangoma --enable-shared --with-ssl --disable-resample --disable-video --disable-opencore-amr --disable-speex-codec --disable-speex-aec --disable-bcg729 --disable-gsm-codec --disable-ilbc-codec --disable-l16-codec --disable-g711-codec --disable-g722-codec --disable-g7221-codec --disable-opencore-amr --disable-silk --disable-opus --disable-video --disable-v4l2 --disable-sound --disable-ext-sound --disable-sdl --disable-libyuv --disable-ffmpeg --disable-openh264 --disable-ipp --disable-libwebrtc --with-external-pa --with-external-srtp

make dep && make && make install

cd ..
```

### Downloading and patching asterisk:
```bash
wget https://github.com/asterisk/asterisk/releases/download/20.5.0/asterisk-20.5.0.tar.gz
tar xzf asterisk-20.5.0.tar.gz && rm asterisk-20.5.0.tar.gz

patch -p2 --forward --directory=asterisk-20.5.0 <asterisk.patch
```

### Building asterisk:
```bash
cd asterisk-20.5.0/

export PKG_CONFIG_PATH="/opt/homebrew/lib/pkgconfig:/opt/sangoma/lib/pkgconfig"
export CFLAGS="-I/opt/homebrew/include -I/opt/sangoma/include"
export LDFLAGS="-L/opt/homebrew/lib -L/opt/sangoma/lib"

./configure --prefix=/opt/sangoma --without-pjproject-bundled --with-pjproject --without-iodbc --with-unixodbc=/opt/homebrew/opt/unixodbc/lib --with-sqlite3=/opt/homebrew/opt/sqlite/lib

make menuselect
make && make install
```
Note: During make menuselect I disable res_geolocation and res_prometheus as I had some issues with that.

### Install launchd agent (manual):
```bash
cp asterisk.plist ~/Library/LaunchAgents

launchctl start asterisk
```
