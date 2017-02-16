FROM ubuntu:xenial

USER root

# Set the locale
RUN locale-gen en_US.UTF-8 
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en  
ENV LC_ALL en_US.UTF-8
RUN dpkg-reconfigure --frontend noninteractive locales \
  && apt-get update -qqy \
  && apt-get -qqy --no-install-recommends install \
    language-pack-en \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# Fonts
RUN apt-get update -qqy \
  && apt-get -qqy --no-install-recommends install \
    fonts-ipafont-gothic \
    xfonts-100dpi \
    xfonts-75dpi \
    xfonts-cyrillic \
    xfonts-scalable \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# Selenium 
RUN apt-get update -qqy \ 
  && apt-get -qqy --no-install-recommends install chromium-browser xvfb maven openjdk-8-jdk \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# Firefox v40.0.3
RUN apt-get update -qqy \
  && apt-get -qqy --no-install-recommends install wget \
  && wget https://downloads.sourceforge.net/project/ubuntuzilla/mozilla/apt/pool/main/f/firefox-mozilla-build/firefox-mozilla-build_40.0.3-0ubuntu1_amd64.deb \
  && dpkg -i firefox-mozilla-build_40.0.3-0ubuntu1_amd64.deb \
  && rm firefox-mozilla-build_40.0.3-0ubuntu1_amd64.deb \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

