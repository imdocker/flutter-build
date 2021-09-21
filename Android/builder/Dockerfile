FROM ubuntu:20.04

ENV UID=1000
ENV GID=1000
ENV USER="developer"
ENV JAVA_VERSION="8"
ENV ANDROID_TOOLS_URL="https://dl.google.com/android/repository/commandlinetools-linux-7583922_latest.zip"
ENV ANDROID_VERSION="29"
ENV ANDROID_BUILD_TOOLS_VERSION="30.0.3"
ENV ANDROID_BUILD_TOOLS2_VERSION="29.0.3"
ENV ANDROID_BUILD_TOOLS3_VERSION="29.0.2"
ENV ANDROID_ARCHITECTURE="x86_64"
ENV ANDROID_SDK_ROOT="/home/$USER/android"
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_VERSION="2.5.1"
ENV FLUTTER_URL="https://storage.googleapis.com/flutter_infra_release/releases/$FLUTTER_CHANNEL/linux/flutter_linux_$FLUTTER_VERSION-$FLUTTER_CHANNEL.tar.xz"
ENV FLUTTER_HOME="/home/$USER/flutter"
ENV FLUTTER_WEB_PORT="8090"
ENV FLUTTER_DEBUG_PORT="42000"
ENV FLUTTER_EMULATOR_NAME="flutter_emulator"
ENV PATH="$ANDROID_SDK_ROOT/cmdline-tools/tools/bin:$ANDROID_SDK_ROOT/emulator:$ANDROID_SDK_ROOT/platform-tools:$ANDROID_SDK_ROOT/platforms:$FLUTTER_HOME/bin:$PATH"

# install all dependencies
ENV DEBIAN_FRONTEND="noninteractive"
RUN apt-get update \
  && apt-get install --yes --no-install-recommends openjdk-$JAVA_VERSION-jdk curl unzip sed git bash xz-utils libglvnd0 ssh sudo \
  && rm -rf /var/lib/{apt,dpkg,cache,log}

# create user
RUN groupadd --gid $GID $USER \
  && useradd -s /bin/bash --uid $UID --gid $GID -m $USER \
  && echo $USER ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USER \
  && chmod 0440 /etc/sudoers.d/$USER

USER $USER
WORKDIR /home/$USER

# android sdk
RUN mkdir -p $ANDROID_SDK_ROOT \
  && mkdir -p /home/$USER/.android \
  && touch /home/$USER/.android/repositories.cfg \
  && curl -o android_tools.zip $ANDROID_TOOLS_URL \
  && unzip -qq -d "$ANDROID_SDK_ROOT" android_tools.zip \
  && rm android_tools.zip \
  && mkdir -p $ANDROID_SDK_ROOT/cmdline-tools/tools \
  && mv $ANDROID_SDK_ROOT/cmdline-tools/bin $ANDROID_SDK_ROOT/cmdline-tools/tools \
  && mv $ANDROID_SDK_ROOT/cmdline-tools/lib $ANDROID_SDK_ROOT/cmdline-tools/tools \
  && echo 1 \
  && yes "y" | sdkmanager "build-tools;$ANDROID_BUILD_TOOLS_VERSION" \
  && echo 2 \
  && yes "y" | sdkmanager "build-tools;$ANDROID_BUILD_TOOLS2_VERSION" \
  && echo 3 \
  && yes "y" | sdkmanager "build-tools;$ANDROID_BUILD_TOOLS3_VERSION" \
  && echo 4 \
  && yes "y" | sdkmanager "platforms;android-$ANDROID_VERSION" \
  && echo 5 \
  && yes "y" | sdkmanager "platform-tools" \
  && echo 6 \
  && yes "y" | sdkmanager "cmdline-tools;4.0"
# flutter

RUN echo $FLUTTER_URL && curl -o flutter.tar.xz $FLUTTER_URL \
  && mkdir -p $FLUTTER_HOME \
  && tar xf flutter.tar.xz -C /home/$USER \
  && rm flutter.tar.xz \
  && echo config && flutter config --no-analytics \
  && echo precache && flutter precache \
  && echo android &&  yes "y" | echo flutter doctor --android-licenses \
  && flutter doctor \
  && echo flutter emulators --create \
  && flutter update-packages

COPY entrypoint.sh /usr/local/bin/
COPY chown.sh /usr/local/bin/
COPY flutter-android-emulator.sh /usr/local/bin/flutter-android-emulator

#ENTRYPOINT [ "/usr/local/bin/entrypoint.sh" ]
