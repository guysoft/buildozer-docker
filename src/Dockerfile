FROM ubuntu:20.04
MAINTAINER Guy Sheffer <guysoft@gmail.com>

ENV SDK_TOOLS="https://dl.google.com/android/repository/platform-tools_r34.0.4-linux.zip"
ENV NDK_DL="https://dl.google.com/android/repository/android-ndk-r17c-linux-x86_64.zip"
ENV NDKVER=r17c
ENV NDKDIR=/ndk/
ENV NDKAPI=21
ENV ANDROIDAPI=28
ENV PIP=pip3


# Parts taken from https://github.com/elgalu/docker-selenium/blob/master/Dockerfile


ENV LANG_WHICH en
ENV LANG_WHERE US
ENV ENCODING UTF-8
ENV LANGUAGE ${LANG_WHICH}_${LANG_WHERE}.${ENCODING}
ENV LANG ${LANGUAGE}
ARG DEBIAN_FRONTEND=noninteractive

# Basic image upgrade:
RUN apt update --fix-missing && apt upgrade -y

# Install base packages
# RUN apt update && apt install -y zip python3 python-pip python python3-virtualenv python-virtualenv python3-pip curl wget lbzip2 bsdtar && dpkg --add-architecture i386 && apt update && apt install -y build-essential libstdc++6:i386 zlib1g-dev zlib1g:i386 openjdk-8-jdk libncurses5:i386 && apt install -y libtool automake autoconf unzip pkg-config git ant gradle rsync

RUN set -x \
    && apt-get update && apt-get install -y \
        build-essential \
        curl \
        git \
        wget \
        p7zip-full \
        python3 \
        python3-distutils \
        python3-dev \
        sudo \
        git \
        zip \
        unzip \
        openjdk-17-jdk \
        autoconf \
        libtool \
        pkg-config \
        zlib1g-dev \
        libncurses5-dev \
        libncursesw5-dev \
        libtinfo5 \
        cmake \
        libffi-dev \
        libssl-dev \
  && rm -rf /var/lib/apt/lists/* \
  && apt -qyy clean
    
#========================================
# Add normal user with passwordless sudo
#========================================
# Layer size: tiny: 0.3 MB
RUN useradd ubuntu \
         --shell /bin/bash  \
         --create-home \
  && usermod -a -G sudo ubuntu \
  && gpasswd -a ubuntu video \
  && echo 'ubuntu:secret' | chpasswd \
&& echo 'ALL ALL = (ALL) NOPASSWD: ALL' >> /etc/sudoers

RUN wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py -O - | python3
RUN pip3 install --upgrade Cython==0.29.33
RUN pip3 install --upgrade buildozer


# download and install Android SDK
# https://developer.android.com/studio#command-line-tools-only
# Source: https://github.com/thyrlian/AndroidSDK/blob/master/android-sdk/Dockerfile
ARG ANDROID_SDK_VERSION=9477386
ENV ANDROID_HOME /opt/android-sdk
RUN mkdir -p ${ANDROID_HOME}/cmdline-tools && \
    wget -q https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_VERSION}_latest.zip && \
    unzip *tools*linux*.zip -d ${ANDROID_HOME}/cmdline-tools && \
    mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/tools && \
    rm *tools*linux*.zip

# set the environment variables
# ENV JAVA_HOME /usr/lib/jvm/java-${JDK_VERSION}-openjdk-amd64
# ENV GRADLE_HOME /opt/gradle
# ENV KOTLIN_HOME /opt/kotlinc
# ENV PATH ${PATH}:${GRADLE_HOME}/bin:${KOTLIN_HOME}/bin:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/cmdline-tools/tools/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator
# WORKAROUND: for issue https://issuetracker.google.com/issues/37137213
ENV LD_LIBRARY_PATH ${ANDROID_HOME}/emulator/lib64:${ANDROID_HOME}/emulator/lib64/qt/lib
# patch emulator issue: Running as root without --no-sandbox is not supported. See https://crbug.com/638180.
# https://doc.qt.io/qt-5/qtwebengine-platform-notes.html#sandboxing-support
ENV QTWEBENGINE_DISABLE_SANDBOX 1

# accept the license agreements of the SDK components
ADD license_accepter.sh /opt/
RUN chmod +x /opt/license_accepter.sh && /opt/license_accepter.sh $ANDROID_HOME



# Install Android SDK:
#RUN mkdir /sdk-install/
#RUN cd /sdk-install && wget https://dl.google.com/android/repository/${SDK_TOOLS}
#RUN cd /sdk-install && unzip ./sdk-tools-*.zip && chmod +x ./tools//bin/sdkmanager
#RUN yes | /sdk-install/tools/bin/sdkmanager --licenses
#RUN /sdk-install/tools/bin/sdkmanager --update
#mRUN /sdk-install/tools/bin/sdkmanager "platform-tools" "platforms;android-28" "build-tools;28.0.3"

# Obtain Android NDK:
RUN mkdir -p /tmp/ndk/ && cd /tmp/ndk/ && wget ${NDK_DL} && unzip -q android-ndk*.zip && mv android-*/ /ndk/


# Install p4a:
RUN $PIP install -U 'https://github.com/kivy/python-for-android/archive/master.zip'  # p4a build 3c1e1ecd-50af-4f17-97e8-002b08a2abe5

# Get the kivy test app:
#RUN mkdir -p /tmp/test-app/ && cd /tmp/test-app && git clone https://github.com/kivy/python-for-android/ .
#RUN cp -R /tmp/test-app/testapps/testapp_flask/ /root/testapp-webview-flask/
#RUN cp -R /tmp/test-app/testapps/testapp_nogui/ /root/testapp-service_only-nogui/

# Prepare output folder:
RUN mkdir /root/output
VOLUME /root/output

# Workspace folder (if used, otherwise the following line will be blank):
VOLUME /root/workspace/

# Start directory:
WORKDIR /root/

# Final command line preparation:
RUN echo 'bash' > /tmp/launchcmd.txt
RUN echo '#!/usr/bin/python3\n\
import json\n\
import os\n\
print("echo \"\"")\n\
print("echo \"To build a kivy demo app, use this command:\"")\n\
if os.environ["PIP"] == "pip2":\n\
    print("echo \"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python2 --private .\"")\n\
    print("shopt -s expand_aliases")\n\
    print("alias testbuild=\"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python2 --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_webview=\"cd ~/testapp-webview-flask && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap webview --requirements=python2,flask --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_service_only=\"cd ~/testapp-service_only-nogui && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap service_only --requirements=pyjnius,python2 --private . && cp *.apk ~/output\"")\n\
else:\n\
    print("echo \"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python3 --private .\"")\n\
    print("shopt -s expand_aliases")\n\
    print("alias testbuild=\"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python3 --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_webview=\"cd ~/testapp-webview-flask && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap webview --requirements=python3,flask --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_service_only=\"cd ~/testapp-service_only-nogui && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap service_only --requirements=pyjnius,python3 --private . && cp *.apk ~/output\"")\n\
with open("/tmp/launchcmd.txt", "r") as f:\n\
    import shlex\n\
    args = shlex.split(f.read().strip())\n\
    print("CMD=()")\n\
    i = -1\n\
    for arg in args:\n\
        i += 1\n\
        print("CMD[" + str(i) + "]=" + shlex.quote(arg))\n\
print("export ANDROIDAP='$ANDROIDAPI'" +\n\
    " ANDROIDNDKVER='$NDKVER'" +\n\
    " NDKAPI='$NDKAPI'" +\n\
    " GRADLE_OPTS=\"-Xms1724m -Xmx5048m -Dorg.gradle.jvmargs='"'"'-Xms1724m -Xmx5048m'"'"'\""+\n\
    " JAVA_OPTS=\"-Xms1724m -Xmx5048m\"" +\n\
    " ANDROIDSDK=/sdk-install/ ANDROIDNDK=\"'$NDKDIR'\"")' > /tmp/cmdline.py

CMD ["bash", "-c", "python3 /tmp/cmdline.py > /tmp/launch.sh; source /tmp/launch.sh; exec \"${CMD[@]}\""]




