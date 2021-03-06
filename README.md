## {OpenGrok installation HOWTO

Requirements:

- Latest [Java](http://www.oracle.com/technetwork/java/) 1.8

- A servlet container like [GlassFish](https://javaee.github.io/glassfish/) or [Tomcat](http://tomcat.apache.org/) ([8.x](https://tomcat.apache.org/download-80.cgi) or later) also running with Java at least 1.8

- [Universal ctags](https://github.com/universal-ctags/ctags) ([Exuberant ctags](http://ctags.sourceforge.net/) work too)

- Python 3

<br>

tomcat root page: `/usr/local/apache-tomcat-8.0/webapps/ROOT`

{OpenGrok logo: `/usr/local/apache-tomcat-8.0/webapps/source/default/img/icon.png`

**NOTE**: those files owned by www:www

## Patches

```
server.xml.patch:   /usr/local/apache-tomcat-8.0/conf

OpenGrok.patch:     /var/opengrok/bin

web.xml.patch:      /var/opengrok/web/source/WEB-INF

index.jsp.patch:    /usr/local/apache-tomcat-8.0/webapps/source
```

## Index

```shell
/var/opengrok/bin/OpenGrok index
```

<br>

## Install in CentOS 7

The script should be run by `root` user

### Update system

```shell
yum -y update
```

### [Install python 3](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-centos-7)

```shell
yum -y group install "Development Tools"

yum -y install https://centos7.iuscommunity.org/ius-release.rpm
yum -y install python36u
yum -y install python36u-pip

pip3.6 install --upgrade pip
```

### [Install JDK 1.8](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora#install-openjdk-8)

```shell
yum -y install java-1.8.0-openjdk-devel
```

### [Install tomcat 8](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-centos-7)

```shell
yum -y install vim
yum -y install wget
wget http://mirror.csclub.uwaterloo.ca/apache/tomcat/tomcat-8/v8.5.38/bin/apache-tomcat-8.5.38.tar.gz

mkdir /opt/tomcat
tar xvf apache-tomcat-8*tar.gz -C /opt/tomcat --strip-components=1

groupadd tomcat
useradd -M -s /bin/nologin -g tomcat -d /opt/tomcat tomcat

chgrp -R tomcat /opt/tomcat

cd /opt/tomcat
chmod -R g+r conf
chmod g+x conf

chown -R tomcat webapps/ work/ temp/ logs/
```

### Install `systemd` unit file

`vim /etc/systemd/system/tomcat.service`:

Note the `CATALINA_OPTS` option

```xml
# Systemd unit file for tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms256M -Xmx2048M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

```

<br>

### Test tomcat

```shell
systemctl daemon-reload
systemctl start tomcat
systemctl enable tomcat
systemctl status tomcat

systemctl restart tomcat
```

Access default tomcat page: `http://SERVER_IP_ADDRESS:8080`

### [Install `universial-ctags`](https://askubuntu.com/questions/796408/installing-and-using-universal-ctags-instead-of-exuberant-ctags)

```shell
git clone https://github.com/universal-ctags/ctags
cd ctags
./autogen.sh
./configure
make
make install

ctags --version
```

<br>

### Install `{OpenGrok`

```shell
cd /opt
wget https://github.com/oracle/opengrok/releases/download/1.2.1/opengrok-1.2.1.tar.gz
tar xvf opengrok-1.2.1.tar.gz
ln -s opengrok-1.2.1 opengrok

cd opengrok
mkdir -p etc src data
```

### Install `opengrok-tools`

```shell
cd /opt/opengrok/src
git clone https://github.com/libuv/libuv

cd /opt/opengrok
python3.6 -m venv opengrok-tools
opengrok-tools/bin/python -m pip install tools/opengrok-tools.tar.gz
```

### Deploy & Index

```shell
cd /opt/opengrok

# -c option should use absolute path
# it'll write into /opt/tomcat/webapps/source/WEB-INF/web.xml
opengrok-tools/bin/opengrok-deploy -c /opt/opengrok/etc/configuration.xml \
	lib/source.war /opt/tomcat/webapps

opengrok-tools/bin/opengrok-indexer \
	-J=-Djava.util.logging.config.file=/opt/opengrok/doc/logging.properties \
	-a lib/opengrok.jar -- \
	-s src -d data -H -P -S -G \
	-W etc/configuration.xml -U http://localhost:8080/source
```

Refresh your web browser grok page, to see latest changes.

<br>

### Miscellaneous

#### Change timezone

```shell
timedatectl set-timezone Asia/Shanghai
```

#### [Configure Tomcat to Listen to Port 80](https://docs.lucee.org/guides/installing-lucee/lucee-server-adminstration-linux/configure-Tomcat-listen-to-port.html)

```shell
# Forward 80 to 8080, thus you don't have to change server.xml
iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# List NAT-PREROUTING rules
iptables -t nat -v -L PREROUTING -n --line-number

# Delete NAT-PREROUTING rule at index 1
iptables -t nat -D PREROUTING 1
```

**NOTE**: above solution is not recommended

Or alternatively, Install [authbind](https://aaronsilber.me/2016/04/24/install-authbind-on-centos-7-x86_64-download-the-rpm/) on CentOS

```shell
rpm -Uvh https://s3.amazonaws.com/aaronsilber/public/authbind-2.1.1-0.1.x86_64.rpm

touch /etc/authbind/byport/80
chmod 500 /etc/authbind/byport/80
chown tomcat /etc/authbind/byport/80
ls -l /etc/authbind/byport/80
```

<br>

`vim /opt/tomcat/bin/startup.sh`:

Change from `exec "$PRGDIR"/"$EXECUTABLE" start "$@"`

to `exec authbind --deep "$PRGDIR"/"$EXECUTABLE" start "$@"`

<br>

`vim /opt/tomcat/conf/server.xml`:

Change `<Connector port="8080" ...` to `<Connector port="80" ...`

<br>

```
systemctl restart tomcat
```

<br>

## References

[How to setup OpenGrok](https://github.com/oracle/opengrok/wiki/How-to-setup-OpenGrok)

[OpenGrok github repository](https://github.com/oracle/opengrok)

[Install {OpenGrok on FreeBSD](https://wiki.bsdforen.de/wiki:marduk:opengrok)

[Using {OpenGrok to boost up source code reading](http://junkman.cn/p/18/03_opengrok.html)

[How To Install Python 3 and Set Up a Local Programming Environment on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-centos-7)

[{OpenGrok Installations](https://github.com/oracle/opengrok/wiki/Installations)

[OpenGrokによるソースコード検索環境の構築](https://qiita.com/vmmhypervisor/items/23e9ea60863c15014836)

[How To Install Java on CentOS and Fedora #Install OpenJDK 8](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora#install-openjdk-8)

[ How To Install Apache Tomcat 8 on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-centos-7)

[universal-ctags Building with configure (*nix including GNU/Linux)](https://github.com/universal-ctags/ctags/blob/master/docs/autotools.rst)

[Install and Configure Tomcat 8 on Centos-7](https://hostpresto.com/community/tutorials/install-and-configure-tomcat-8-on-centos-7)

[INSTALL TOMCAT 8](https://snapdev.net/2016/09/29/install-tomcat-8-on-centos-7/)

<br>

*Created 180414 Rev_no: 2*
