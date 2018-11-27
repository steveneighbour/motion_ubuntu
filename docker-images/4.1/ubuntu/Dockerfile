FROM        ubuntu:16.04

#RUN         mkdir /tmp/workdir
#WORKDIR     /tmp/workdir

RUN     apt-get update && \
        apt-get install -y --no-install-recommends ca-certificates expat libgomp1 git unzip \
	autoconf automake cmake curl bzip2 libexpat1-dev g++ gcc gperf libtool make nasm perl \
        pkg-config python libssl-dev yasm libfreetype6-dev zlib1g-dev 
	
#ARG        PKG_CONFIG_PATH=/opt/ffmpeg/lib/pkgconfig
#ARG        LD_LIBRARY_PATH=/opt/ffmpeg/lib
#ARG        MAKEFLAGS="-j4"

ENV         FFMPEG_VERSION=4.1     \
            FDKAAC_VERSION=0.1.5      \
            LAME_VERSION=3.99.5       \
            LIBASS_VERSION=0.13.7     \
            OPENCOREAMR_VERSION=0.1.5 \
            OPENJPEG_VERSION=2.1.2    \
            VPX_VERSION=1.7.0         \
            X264_VERSION=20170226-2245-stable \
            X265_VERSION=2.3          \
            FREETYPE_VERSION=2.5.5    \
            FRIBIDI_VERSION=0.19.7    \
            FONTCONFIG_VERSION=2.12.4 \
            LIBVIDSTAB_VERSION=1.1.0  \
            KVAZAAR_VERSION=1.2.0     \
            AOM_VERSION=v1.0.0        \
            SRC=/usr/local

## x264 http://www.videolan.org/developers/x264.html
RUN     DIR=/tmp/x264 && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://download.videolan.org/pub/videolan/x264/snapshots/x264-snapshot-${X264_VERSION}.tar.bz2 | \
        tar -jx --strip-components=1 && \
        ./configure --prefix=/opt/ffmpeg --enable-shared --enable-pic --disable-cli && \
        make && \
        make install 
	
### x265 http://x265.org/
RUN     DIR=/tmp/x265 && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://download.videolan.org/pub/videolan/x265/x265_${X265_VERSION}.tar.gz  | \
        tar -zx && \
        cd x265_${X265_VERSION}/build/linux && \
	sed -i "/-DEXTRA_LIB/ s/$/ -DCMAKE_INSTALL_PREFIX=\/opt/ffmpeg//" multilib.sh && \
        sed -i "/^cmake/ s/$/ -DENABLE_CLI=OFF/" multilib.sh && \
        ./multilib.sh && \
        make -C 8bit install

### libmp3lame http://lame.sourceforge.net/
RUN     DIR=/tmp/lame && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://kent.dl.sourceforge.net/project/lame/lame/$(echo ${LAME_VERSION} | sed -e 's/[^0-9]*\([0-9]*\)[.]\([0-9]*\)[.]\([0-9]*\)\([0-9A-Za-z-]*\)/\1.\2/')/lame-${LAME_VERSION}.tar.gz | \
        tar -zx --strip-components=1 && \
        ./configure --prefix=/opt/ffmpeg --bindir=/opt/ffmpeg/bin --enable-shared --enable-nasm --enable-pic --disable-frontend && \
        make && \
        make install

RUN     DIR=/tmp/fdk-aac && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://github.com/mstorsjo/fdk-aac/archive/v${FDKAAC_VERSION}.tar.gz | \
        tar -zx --strip-components=1 && \
        autoreconf -fiv && \
        ./configure --prefix=/opt/ffmpeg --enable-shared --datadir="${DIR}" && \
        make && \
        make install
	
## openjpeg https://github.com/uclouvain/openjpeg
RUN     DIR=/tmp/openjpeg && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://github.com/uclouvain/openjpeg/archive/v${OPENJPEG_VERSION}.tar.gz | \
        tar -zx --strip-components=1 && \
        cmake -DBUILD_THIRDPARTY:BOOL=ON -DCMAKE_INSTALL_PREFIX=/opt/ffmpeg . && \
        make && \
        make install

## fontconfig https://www.freedesktop.org/wiki/Software/fontconfig/
RUN     DIR=/tmp/fontconfig && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sLO https://www.freedesktop.org/software/fontconfig/release/fontconfig-${FONTCONFIG_VERSION}.tar.bz2 &&\
        tar -jx --strip-components=1 -f fontconfig-${FONTCONFIG_VERSION}.tar.bz2 && \
        ./configure -prefix="${PREFIX}" --disable-static --enable-shared && \
        make && \
        make install 

## ffmpeg https://ffmpeg.org/
RUN     DIR=/tmp/ffmpeg && mkdir -p ${DIR} && cd ${DIR} && \
        curl -sLO https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
        tar -jx --strip-components=1 -f ffmpeg-${FFMPEG_VERSION}.tar.bz2

RUN     DIR=/tmp/ffmpeg && mkdir -p ${DIR} && cd ${DIR} && \
        ./configure \
        --disable-debug \
        --disable-doc \
        --disable-ffplay \
        --enable-shared \
        --enable-avresample \
        --enable-gpl \
        --enable-libfreetype \
        --enable-libmp3lame \
        --enable-libopenjpeg \
        --enable-libx265 \
        --enable-libx264 \
        --enable-nonfree \
        --enable-openssl \
        --enable-libfdk_aac \
        --enable-postproc \
        --enable-small \
        --enable-version3 \
        --extra-cflags="-I${PREFIX}/include" \
        --extra-ldflags="-L${PREFIX}/lib" \
        --extra-libs=-ldl \
        --prefix="${PREFIX}" && \
        make && \
        make install && \
        make distclean && \
        hash -r && \
        cd tools && \
        make qt-faststart && \
        cp qt-faststart ${PREFIX}/bin

## cleanup
RUN     ldd ${PREFIX}/bin/ffmpeg | grep opt/ffmpeg | cut -d ' ' -f 3 | xargs -i cp {} /usr/local/lib/ && \
        cp ${PREFIX}/bin/* /usr/local/bin/ && \
        cp -r ${PREFIX}/share/ffmpeg /usr/local/share/ && \
	rm -fr /tmp/* && \
        LD_LIBRARY_PATH=/usr/local/lib ffmpeg -buildconf

ENV     LD_LIBRARY_PATH=/usr/local/lib

RUN git clone https://github.com/Motion-Project/motion.git  && \
   cd motion  && \
   autoreconf -fiv && \
   ./configure && \
   make && \
   make install && \
   cd .. && \
   rm -fr motion && \

CMD [ "motion", "-n" ]
