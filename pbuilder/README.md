# pbuilder环境打包

pbuilder代表Personal Builder，它是一个Debian系软件包构建环境。旨在为每一个软件包的构建提供一个干净且相同的编译环境。

### Table of Contents

- [安装pbuilder](#安装pbuilder)
- [构建一个pbuilder镜像](#构建一个pbuilder镜像)
- [更新pbuilder镜像](#更新pbuilder镜像)
- [使用pbuilder镜像打包](#使用pbuilder镜像打包)
- [参数配置](#参数配置)
- [常见错误](#常见错误)

### 安装pbuilder

```shell
$ sudo apt-get install pbuilder debootstrap devscripts dh-make
```

### 构建一个pbuilder镜像

生成一个名称为buster-amd64.tgz的pbuilder镜像, 如果这里构建失败可以参考 - [参数配置](#参数配置)

```shell
$ sudo pbuilder create --distribution unstable  --debootstrapopts --variant=buildd   --basetgz buster-amd64.tgz --debootstrapopts --no-check-gpg --debootstrapopts --merged-usr
```

参数解释：

    --distribution    #指定仓库codename
    --debootstrapopts #指定debootstrap命令的参数，每一个参数都需要使用--debootstrapopts进行指定
    --merged-usr      #是debootstrap的参数，指定/bin文件是一个软连接指向/usr/bin

### 更新pbuilder镜像

使用--save-after-login参数将保存login到pbuilder镜像后的操作。

```shell
$ sudo pbuilder --login --basetgz buster-amd64.tgz --save-after-login 
```

参数解释:
    
    --login               #登陆到pbuilder镜像
    --save-after-login    #在退出pbuilder镜像的时候，将保存login之后的所有操作

### 使用pbuilder镜像打包

方案一：使用dsc源码文件进行直接打包构建。

```shell
$ sudo pbuilder --build  --logfile log.txt --basetgz buster-amd64.tgz --allow-untrusted --hookdir /var/cache/pbuilder/hooks --use-network yes --aptcache "" --buildresult . --debbuildopts -sa *.dsc
```

方案二：使用--login登陆到pbuilder环境后借助其他工具进行打包，--bindmounts参数绑定主机目录挂载到pbuilder环境内。

```shell
$ sudo pbuilder --login --basetgz buster-amd64.tgz --bindmounts sourceDir/
```

### 参数配置

配置文件/etc/pbuilderrc，配置pbuilder环境的源仓库、以及pbuilder环境拉取时额外需要增加的一些软件包等参数:
  
    # this is your configuration file for pbuilder.
    # the file in /usr/share/pbuilder/pbuilderrc is the default template.
    # /etc/pbuilderrc is the one meant for overwriting defaults in
    # the default template
    #
    # read pbuilderrc.5 document for notes on specific options.
    MIRRORSITE=https://mirrors.tuna.tsinghua.edu.cn/debian/
    DEBOOTSTRAPOPTS=(
      "--no-check-gpg"
      "--variant=buildd"
      "--include=debian-keyring,git,pbuilder,devscripts,perl-openssl-defaults,sudo"
    )	

修改debootstrap脚本/usr/sbin/debootstrap:
    
    ###########################################################################
    if [ -n "$DISABLE_KEYRING" ] && [ -n "$FORCE_KEYRING" ]; then
    #       error 1 BADARG "Both --no-check-gpg and --force-check-gpg specified, please pick one (at most)"
            echo ""
    fi

    ###########################################################################

修改pbuilder脚本，解决在安装额外软件包时需要交互的问题，/usr/lib/pbuilder/pbuilder-createbuildenv:
    
    $TRAP umountproc_cleanbuildplace_trap exit sighup
    $CHROOTEXEC apt-get -q "${APTGETOPT[@]}" --allow-insecure-repositories update

    install_packages_for_optional_features

    if [ -n "$REMOVEPACKAGES" ]; then remove_packages $REMOVEPACKAGES ; fi
    recover_aptcache
    $CHROOTEXEC apt-get -q -y "${APTGETOPT[@]}" "${FORCE_CONFNEW[@]}" --allow-unauthenticated dist-upgrade
    $CHROOTEXEC apt-get -q -y "${APTGETOPT[@]}" --allow-unauthenticated install \
      build-essential \
      dpkg-dev \
      $EXTRAPACKAGES
      save_aptcache

### 常见错误

错误一：

     E: No such script: /usr/share/debootstrap/scripts/eagle 
     E: debootstrap failed 
     E: debootstrap.log not present 
     W: Aborting with an error

解决方案:

```shell
$ cp /usr/share/debootstrap/scripts/sid /usr/share/debootstrap/scripts/eagle
```
