# ###########################################################################################
# Basic docker build of SDL_Core https://github.com/smartdevicelink/sdl_core
# ###########################################################################################

# This image is based of debian with git preinstalled
FROM buildpack-deps:16.04-scm

# MAINTAINER Corey Maylone version:1.0

ENV CORE_BRANCH=v4.4.1-global-log4cxx-apr \
    CORE_NETWORK_INTERFACE=eth0 \
    HMI_WEBSOCKET_ADDRESS=127.0.0.1:8087 \
    CORE_CWD=/tmp/sdl \
    HMI_CWD=/var/www \
    SDL_CORE_GIT=https://github.com/SlackSeven/sdl_core

# Create our working directory and Download SDL Core from github
RUN git clone -b $CORE_BRANCH $SDL_CORE_GIT $CORE_CWD   
    
# Download SDL HMI from github that correlates to the current SDL Core branch
# RUN git clone -b $CORE_BRANCH https://github.com/smartdevicelink/sdl_hmi.git $HMI_CWD

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \ 
        cmake \
        pkg-config \
        libudev-dev \
        libsqlite3-dev \
        libssl-dev \
        nginx \
        # python \
        # sudo \
        libapr1-dev \
        libaprutil1-dev \
        liblog4cxx-dev \
        supervisor \
        sqlite3 \
    && rm -rf /var/lib/apt/lists/*

# Change directory to the unpacked SDL Core sourcecode
WORKDIR $CORE_CWD

# Generate our Makefile, compile it and install
RUN cmake -DBUILD_AVAHI_SUPPORT=OFF -DBUILD_USB_SUPPORT=OFF -DBUILD_BT_SUPPORT=OFF -DEXTENDED_MEDIA_MODE=OFF -DENABLE_HMI_PTU_DECRYPTION=OFF DENABLE_SECURITY=OFF \
    -DUSE_COTIRE=OFF \
    -DTELEMETRY_MONITOR=OFF \
    -DNO_REBUILD_3RD_PARTY=ON \
    -DCMAKE_INSTALL_PREFIX=/usr \
    . \
    && make \
    && make install \
    && cd / && rm -rf $CORE_CWD

# Configure dynamic linker run-time bindings
RUN ldconfig

# Change directory to where the binaries are stored
WORKDIR /usr/bin

# Copy SDL Core start script into container
COPY setup.sh setup.sh

# Give the set-up script execute permissions
RUN chmod +x setup.sh

# Delete default nginx configuration
RUN rm -v /etc/nginx/nginx.conf

# Copy our new configuration
COPY nginx.conf /etc/nginx/

# Copy supervisord config
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Expose the following ports
#   3001: Exposes Core's file system
#   8087:  Expose SDL Core's Web Socket port. Used to comunicate with the HMI
#   8080:  Returns index.html of HMI Core
#   12345: Expose SDL Core's primary port. Used to communicate with the SDL Core instance over TCP
EXPOSE 3001 8080 8087 12345 

# Run the start script that will launch SDL Core!
ENTRYPOINT $PWD/setup.sh