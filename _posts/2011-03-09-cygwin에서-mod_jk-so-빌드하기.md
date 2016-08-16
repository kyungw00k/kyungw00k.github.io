---
layout: post
title: Cygwin에서 mod_jk.so 빌드하기
date: 2011-03-09 14:34:28.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
tags:
- Unix
- Cygwin
- mod_jk
---

Cygwin 환경에서 apache2 설치 후, mod_jk.so(tomcat-connectors-1.2.31)를 빌드해서 tomcat과 같이 쓰려는데 한방에 되지 않았다.

일단 무슨에러가 나는지 부터 살펴본 후, 하나씩 하나씩 해결해 보도록 하겠다.

압축을 푼 후,

`./configure` 하면 아래와 같이 에러가 뜬다.

![](/images/2011/03/09/cygwin_modjk_1.png)

웹서버를 찾을 수 없다는 메시지가 뜬다.  이와 관련되서 구글링을 한다면 분명 "apache인 경우에는 apxs 또는 apxs2 위치를 지정하면 해결된다" 라고 나온다.

이때는 다음과 같이 입력하면 위의 configure는 통과한다.

`./configure --with-apxs=/usr/sbin/apxs2`

![](/images/2011/03/09/cygwin_modjk_2.png)

`configure` 단계가 통과했으니, 이제 `make`를 해보자.

빌드를 한 후, `make install` 하면 `mod_jk.so`가 없다고 어쩌고 저쩌고 한다.

![](/images/2011/03/09/cygwin_modjk_4.png)

그래서 `native/apache-2.0` 로 들어가보니 `mod_jk.so`가 없었다.
(필자의 경우 apache2를 설치했기 때문에 apache-2.0에서 확인을 했다)

구글링해서 걸린 URL중 [apache mail archive](http://mail-archives.apache.org/mod_mbox/tomcat-users/200802.mbox/%3C47A8AF63.8040201@kippdata.de%3E)를 보니까 "apache2.0/Makefile에 뭔가 빠져서 안된다는 내용이있고, **어쩌고저쩌고 해서 수정하면 된다**" 라고 나와있길래 `make clean` 후 다시 make 해보았다.

![](/images/2011/03/09/cygwin_modjk_5.png)

오류가 또 났다. `/usr/lib/libdb-4.2.la`가 없덴다. 이런!

그런데 `/usr/lib/libdb-4.5.la`가 있었다! 하여 symbolic link를 걸고 다시 make 해보았다.

![](/images/2011/03/09/cygwin_modjk_6.png)

`mod_jk.so` 가 생겼다. 이제 맘편히 `make install` 하면 되겠다.

아래는 `native/apache-2.0/Makefile` 내용이다(82라인이 수정되었다.)

```
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

##

APXS=/usr/sbin/apxs2
OS=Windows_NT
JAVA_HOME=
CP=/usr/bin/cp
APACHE_DIR=/usr
MKDIR=/usr/bin/mkdir
APXSCFLAGS=-O2  -DHAVE_APR  -I/usr/include/apr-1 -I/usr/include/apr-1  -DHAVE_CONFIG_H
APXSCPPFLAGS=-DCYGWIN
APXSLDFLAGS=
CC=gcc

# Defaults
libexecdir=${APACHE_DIR}/modules

JK=../common/

# Defines APACHE_OBJECTS - the list of all common files
include ../common/list.mk

# Apache2 settings, values guessed by Apache config and used to build it
# Will define libexecdir, LIBTOOL, etc
include /usr/share/apache2/build/config_vars.mk

# Local settings ( overriding/appending to Apache's )
COMMON=../common
JK_INCL=-DUSE_APACHE_MD5 -I ${COMMON}
JAVA_INCL=-I ${JAVA_HOME}/include -I ${JAVA_HOME}/include/${OS}
JAVA_LIB=-L ${JAVA_HOME}/jre/lib/${ARCH} -L ${JAVA_HOME}/lib/${ARCH}/native_threads
CFLAGS=-I/usr/include/apache2  -DHAVE_CONFIG_H ${JK_INCL} ${JAVA_INCL} ${APXSCPPFLAGS} ${APXSCFLAGS} ${EXTRA_CFLAGS} ${EXTRA_CPPFLAGS}

# Implicit rules
include ../scripts/build/rules.mk

OEXT=.lo

all: Makefile mod_jk.so
install: install_dynamic

Makefile: Makefile.in
        echo Regenerating Makefile
        ( cd ..; ./config.status )

lib_jk.la: mod_jk.lo ${APACHE_OBJECTS}
        $(LIBTOOL) --mode=link $(CC) -o lib_jk.la -static -rpath ${libexecdir}/jk mod_jk.lo $(APACHE_OBJECTS)

install_static:
        @echo ""
        @echo "Copying files to Apache Modules Directory..."
        -${MKDIR} ${APACHE_DIR}/modules/jk
        ${CP} config.m4 ${APACHE_DIR}/modules/jk
        ${LIBTOOL} --mode=install cp lib_jk.la ${APACHE_DIR}/modules/jk
        @echo ""
        @echo "Please be sure to re-compile Apache..."
        @echo ""
        @echo "cd ${APACHE_DIR}"
        @echo "./buildconf"
        @echo "./configure --with-mod_jk"
        @echo "make"
        @echo ""

#################### Dynamic .so file ####################
# APXS will compile every file, this is derived from apxs

mod_jk.la: mod_jk.lo $(APACHE_OBJECTS)
        $(LIBTOOL) --mode=link ${COMPILE} $(APXSLDFLAGS) -no-undefined `${APXS} -q LDFLAGS` -o $@ -module -rpath ${libexecdir} -avoid-version mod_jk.lo $(APACHE_OBJECTS) /usr/lib/libapr-1.la /usr/lib/libhttpd2core.la /usr/lib/libaprutil-1.la

mod_jk.so: mod_jk.la
        ../scripts/build/instdso.sh SH_LIBTOOL='$(LIBTOOL)' mod_jk.la `pwd`

install_dynamic:
        @echo ""
        @echo "Installing files to Apache Modules Directory..."
        $(APXS) -i mod_jk.la
        @echo ""
        @echo "Please be sure to arrange ${APACHE_DIR}/conf/httpd.conf..."
        @echo ""

clean:
        rm -f *.o *.lo *.a *.la *.so *.so.* *.slo
        rm -rf .libs

maintainer-clean: clean
```
