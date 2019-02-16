## {OpenGrok HOWTO


Requirements:

- Latest [Java](http://www.oracle.com/technetwork/java/) 1.8

- A servlet container like [GlassFish](https://javaee.github.io/glassfish/) or [Tomcat](http://tomcat.apache.org/) (8.x or later) also running with Java at least 1.8

- [Universal ctags](https://github.com/universal-ctags/ctags) ([Exuberant ctags](http://ctags.sourceforge.net/) work too)

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

### Install in Linux

**TODO**

## References

[OpenGrok github repository](https://github.com/oracle/opengrok)

[Install {OpenGrok on FreeBSD](https://wiki.bsdforen.de/wiki:marduk:opengrok)

[Using {OpenGrok to boost up source code reading](http://junkman.cn/p/18/03_opengrok.html)

[{OpenGrok Installations](https://github.com/oracle/opengrok/wiki/Installations)

<br>

*Created 180414 Rev_no: 2*
