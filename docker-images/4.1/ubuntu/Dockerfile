FROM        ubuntu:17.10

WORKDIR     /tmp/workdir

RUN     apt-get update && \
        apt-get install -y --no-install-recommends ca-certificates expat libgomp1 git unzip \
	autoconf automake cmake curl bzip2 libexpat1-dev g++ gcc gperf libtool make nasm perl \
        pkg-config python libssl-dev yasm libfreetype6-dev zlib1g-dev 
	
ARG        PKG_CONFIG_PATH=/opt/ffmpeg/lib/pkgconfig
ARG        LD_LIBRARY_PATH=/opt/ffmpeg/lib
ARG        PREFIX=/opt/ffmpeg
ARG        MAKEFLAGS="-j4"

ENV         FFMPEG_VERSION=4.1     \
            OPENJPEG_VERSION=2.1.2    \
            X264_VERSION=20170226-2245-stable \
            X265_VERSION=2.3          \
            SRC=/usr/local

RUN     DIR=/tmp/x264 && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://download.videolan.org/pub/videolan/x264/snapshots/x264-snapshot-${X264_VERSION}.tar.bz2 | \
        tar -jx --strip-components=1 && \
        ./configure --prefix="${PREFIX}" --enable-shared --enable-pic --disable-cli && \
        make && \
        make install && \
        rm -rf ${DIR}
	
### x265 http://x265.org/
RUN     DIR=/tmp/x265 && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://download.videolan.org/pub/videolan/x265/x265_${X265_VERSION}.tar.gz  | \
        tar -zx && \
        cd x265_${X265_VERSION}/build/linux && \
        sed -i "/-DEXTRA_LIB/ s/$/ -DCMAKE_INSTALL_PREFIX=\${PREFIX}/" multilib.sh && \
        sed -i "/^cmake/ s/$/ -DENABLE_CLI=OFF/" multilib.sh && \
        ./multilib.sh && \
        make -C 8bit install && \
        rm -rf ${DIR}

## openjpeg https://github.com/uclouvain/openjpeg
RUN     DIR=/tmp/openjpeg && \
        mkdir -p ${DIR} && \
        cd ${DIR} && \
        curl -sL https://github.com/uclouvain/openjpeg/archive/v${OPENJPEG_VERSION}.tar.gz | \
        tar -zx --strip-components=1 && \
        cmake -DBUILD_THIRDPARTY:BOOL=ON -DCMAKE_INSTALL_PREFIX="${PREFIX}" . && \
        make && \
        make install && \
        rm -rf ${DIR}
	
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
        --enable-libx265 \
        --enable-libx264 \
        --enable-nonfree \
        --enable-openssl \
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
