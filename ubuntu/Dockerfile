FROM ubuntu:16.04

ENV DEBIAN_FRONTEND noninteractive

COPY ./shiyanlou/ /etc/shiyanlou/

RUN \
    # apt
    sed -i "s/archive.ubuntu.com/mirrors.aliyuncs.com/g" /etc/apt/sources.list; \
    echo 'deb-src http://mirrors.aliyuncs.com/ubuntu/ trusty multiverse' >> /etc/apt/sources.list; \

    # base package
    apt-get update;apt-get -y upgarde;\
    apt-get install -y apt-utils vim openssh-server git zsh xfce4 python-setuptools \
        python-pip wget curl apt-transport-https ca-certificates htop software-properties-common \
        python3-pip supervisor unzip iputils-ping net-tools iproute2 telnet gedit build-essential \
        dnsutils tcpdump binutils sudo lsb-release vim-gtk python3-setuptools openssl tree \
        netcat netcat man locales build-essential; \

    # software
    apt-get install -y xfce4-terminal firefox; \
    apt-get clean; apt-get autoclean; \
    easy_install mr.laforge;

RUN \

    # xrdp depends
    apt-get install -y autoconf libtool pkg-config gcc g++ make libssl-dev libpam0g-dev \
        libjpeg-dev libx11-dev libxfixes-dev libxrandr-dev flex bison libxml2-dev intltool \
        xsltproc xutils-dev python-libxml2 g++ xutils libfuse-dev libmp3lame-dev nasm \
        libpixman-1-dev xserver-xorg-dev; \

    # build xrdp
    cd /etc/shiyanlou/src/ && tar -xf xrdp-0.9.5.tar.gz && cd xrdp-0.9.5 && ./bootstrap \
        && ./configure --enable-fuse --enable-mp3lame \
        --enable-pixman && make && make install && \

    # build X11rdp
    cd /etc/shiyanlou/src/xrdp-0.9.5/xorg/X11R7.6 && ./buildx.sh /opt/X11rdp && \
        ln -s /opt/X11rdp/bin/X11rdp /usr/bin; \

    # build xorgxrdp, refer https://github.com/neutrinolabs/xrdp/wiki/Compiling-and-using-xorgxrdp
    cd /etc/shiyanlou/src/ && tar -xf xorgxrdp-0.2.4.tar.gz && cd xorgxrdp-0.2.4 && ./bootstrap \
        && ./configure && make && make install ;\


    # locale
    apt-get install -y language-pack-gnome-zh-hans language-pack-zh-hans firefox-locale-zh-hans \
        fonts-wqy-zenhei fonts-wqy-microhei xfonts-wqy ttf-ubuntu-font-family; \
    locale-gen zh_CN zh_CN.UTF-8 en_US.UTF-8; dpkg-reconfigure locales; update-locale \
    LANG="zh_CN.UTF-8" LANGUAGE="zh_CN"; \
    echo '# ' > /etc/default/locale; \
    cp /etc/shiyanlou/conf/locale /etc/default/locale; \

    echo 'export LANG="zh_CN.UTF-8"' >> /etc/xdg/xfce4/xinitrc; \
    echo 'export LANGUAGE="zh_CN"' >> /etc/xdg/xfce4/xinitrc; \

    # timezone
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    sed -i 's/UTC=yes/UTC=no/' /etc/default/rcS; \
   
    # supervisor
    mv /etc/supervisor/supervisord.conf /etc/supervisor/supervisord.conf.default; \
    mv /etc/shiyanlou/conf/supervisord.conf /etc/supervisor/supervisord.conf; \
    mv /etc/shiyanlou/conf/supervisor_service.conf /etc/supervisor/conf.d/; \

    # add shiyanlou user
    groupadd -g 5000 -r shiyanlou; \
    useradd -l -m -s /bin/bash -d /home/shiyanlou -u 5000 -g 5000 shiyanlou; \
    su - shiyanlou -c 'mkdir ~/Code'; \
    su - shiyanlou -c 'echo xfce4-session > ~/.xsession'; \

    echo 'shiyanlou:shiyanlou' | chpasswd; \
    echo 'source /etc/environment' >> /home/shiyanlou/.profile; \ 

    echo 'shiyanlou ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/shiyanlou; \
    echo 'Defaults:shiyanlou !requiretty' >> /etc/sudoers.d/shiyanlou; \
    
    su - shiyanlou -c "mkdir -p ~/.config/xfce4"; \
    cp -r /etc/shiyanlou/conf/xfce4/. /home/shiyanlou/.config/xfce4/; \

    su - shiyanlou -c "curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh"; \
    sed -i 's/^ZSH_THEME=.*/ZSH_THEME="philips"/' /home/shiyanlou/.zshrc; \
    echo 'DISABLE_AUTO_UPDATE="true"' >> /home/shiyanlou/.zshrc ;\
    chsh -s /usr/bin/zsh shiyanlou; 

RUN \

    mkdir /home/shiyanlou/Desktop; \

    # ssh
    mkdir /var/run/sshd; \

    # vscode
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg; \
    mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg; \ 
    echo "deb [arch=amd64] http://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list; \
    apt-get update; apt-get install -y code; \
    cp /usr/share/applications/code.desktop /home/shiyanlou/Desktop; \
    chmod +x /home/shiyanlou/Desktop/code.desktop; \


    # firefox 
    cp /etc/shiyanlou/conf/firefox/mozilla.cfg /usr/lib/firefox/; \
    cp /usr/share/applications/firefox.desktop /home/shiyanlou/Desktop; \
    chmod +x /home/shiyanlou/Desktop/firefox.desktop; \


    # chinese input method
    apt-get install -y fcitx fcitx-frontend-gtk2 fcitx-frontend-gtk3 fcitx-module-kimpanel \
        im-config libopencc1 fcitx-libs fcitx-libs-qt libqt4-dbus; \
    dpkg -i /etc/shiyanlou/src/sogoupinyin_2.2.0.0102_amd64.deb; \

    # sublime 
    dpkg -i /etc/shiyanlou/src/sublime-text_build-3083_amd64.deb; \
    apt-get install -y -f; \
    cp /usr/share/applications/sublime_text.desktop /home/shiyanlou/Desktop; \
    chmod +x /home/shiyanlou/Desktop/sublime_text.desktop; \


    # xfce
    cp /usr/share/applications/xfce4-terminal.desktop /home/shiyanlou/Desktop; \
    chmod +x /home/shiyanlou/Desktop/xfce4-terminal.desktop; \


    # gvim   
    cp /usr/share/applications/gvim.desktop /home/shiyanlou/Desktop; \
    chmod +x /home/shiyanlou/Desktop/gvim.desktop; \


    # gedit 
    cp /usr/share/applications/gedit.desktop /home/shiyanlou/Desktop; \
    chmod +x /home/shiyanlou/Desktop/gedit.desktop; \

    chown -R shiyanlou:shiyanlou /home/shiyanlou/Desktop; \

    # disabled auto create user_dir
    sed -i "s/enabled=True/enabled=False/g" /etc/xdg/user-dirs.conf; \ 
    # custom xrdp config
    cp /etc/shiyanlou/conf/xrdp.ini /etc/xrdp/; \
    cp /etc/shiyanlou/conf/sesmin.ini /etc/xrdp/ ;


RUN \
    # oracle java8
    add-apt-repository -y ppa:webupd8team/java; \
    apt-get update; \
    echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections; \
    apt-get install -y oracle-java8-installer; \

    # pip source 
    pip install --upgrade pip; \
    pip3 install --upgrade pip; \
    cp /etc/shiyanlou/conf/pip.conf /home/shiyanlou/.pip; \

    # nodejs 8.x
    curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -; \
    apt-get install -y nodejs g++; \
    echo "registry=https://registry.npm.taobao.org" > /home/shiyanlou/.npmrc; \
    
    # ruby
    apt-get install -y ruby; \
    cp /etc/shiyanlou/conf/gemrc /home/shiyanlou/.gemrc; \

    # php 7.0
    apt-get install -y php; \
    curl -sS https://getcomposer.org/installer | php; \
    mv composer.phar /usr/local/bin/composer; \
    composer config -g repo.packagist composer https://packagist.phpcomposer.com; \
    

    # mysql5.7
    apt-get install -y mysql-server mysql-client mysql-common; \ 
    
    # mongodb-org 3.6
    apt-get install -y mongodb; \
    apt-get remove mongodb && apt-get autoremove; \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5; \
    echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.6.list; \
    apt-get install -y mongodb-org; \
    cp /etc/shiyanlou/conf/mongodb /etc/init.d/mongodb; \
 
 
    # redis 4.0
    add-apt-repository -y ppa:chris-lea/redis-server; \
    apt-get update; apt-get install -y redis-server; \
    sed -i "s/ulimit*/true/g" /etc/init.d/redis-server; \
 
    # web server 
    apt-get install -y apache2 nginx; \


    # vnc 
    apt-get install libtasn1-3-bin libtasn1-bin; \
    dpkg -i /etc/shiyanlou/src/tigervncserver_1.8.0-1ubuntu1_amd64.deb; \
    su - shiyanlou -c 'mkdir ~/.vnc'; \
    su - shiyanlou -c 'echo 123456 | /usr/bin/tigervncpasswd -f > ~/.vnc/passwd'; \
    su - shiyanlou -c 'echo startxfce4 > ~/.vnc/xstartup'; \
    chmod 600 /home/shiyanlou/.vnc/passwd; \
    chmod 755 /home/shiyanlou/.vnc/xstartup; \ 

    # vim's vimrc
    cp /etc/shiyanlou/conf/vimrc /home/shiyanlou/.vimrc; \
    
    # some fix 
    rm /etc/xdg/autostart/pulseaudio.desktop /etc/xdg/autostart/xscreensaver.desktop; \ 
    echo 'rm -rf /home/shiyanlou/thinclient_drives' >> /home/shiyanlou/.zshrc; \

    # change desktop background and pannel icon
    cp /etc/shiyanlou/src/back.png /usr/share/backgrounds/xfce/xfce-teal.jpg; \
    cp /etc/shiyanlou/src/xfce4_shiyanlouicon.png /usr/share/pixmaps/xfce4_shiyanlouicon.png; \

    # clean
    apt-get -y clean; apt-get -y autoremove; \
    chmod a+x /usr/sbin; chmod a+x /usr/local/sbin; chmod a+x /usr/bin; \
    rm -rf /etc/shiyan/conf && rm -rf /etc/shiyanlou/src; 



CMD ["/etc/shiyanlou/sbin/init.sh"]
