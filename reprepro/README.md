# reprepro软件包仓库管理工具

### Table of Contents

- [安装reprepro](#安装reprepro)
- [创建一个reprepro仓库目录](#创建一个reprepro仓库目录)
- [仓库配置](#仓库配置)
- [仓库导入软件包](#仓库导入软件包)
- [仓库删除软件包](#仓库删除软件包)
- [仓库更新](#仓库更新)
- [记一些reprepro其他操作](#记一些reprepro其他操作)

### 安装reprepro

```shell
$ sudo apt install reprepro
```

### 创建一个reprepro仓库目录

```shell
$ mkdir repoTest/
$ cd repoTest/
$ mkdir conf/
```

### 仓库配置

conf/distributions:

    Origin: Linux 
    Label: linux
    Codename: eagle
    Version: 2022
    Architectures: amd64 arm64 source
    Components: main contrib non-free
    Update: eagle-main eagle-contrib eagle-nonfree
    UDebComponents: main
    Contents: percomponent nocompatsymlink .bz2
    SignWith: xxxxx@xxx.com
    Description: Linux packages repo.
    Log: eagle.log

conf/options(可选示例):

    morguedir +o/archives
    logdir +o/log
    verbose

conf/updates(可选示例):

    Name: eagle-main
    Suite: unstable
    Architectures: amd64 arm64 source
    Components: main
    UDebComponents:
    Method: https://mirrors.tuna.tsinghua.edu.cn/debian/
    VerifyRelease: blindtrust
    FilterSrcList: purge ./mirror.packages.main

    Name: eagle-contrib
    Suite: unstable
    Architectures: amd64 arm64 source
    Components: contrib
    UDebComponents:
    Method: https://mirrors.tuna.tsinghua.edu.cn/debian/
    VerifyRelease: blindtrust
    FilterSrcList: purge ./mirror.packages.contrib

    Name: eagle-nonfree
    Suite: unstable
    Architectures: amd64 arm64 source
    Components: nonfree
    UDebComponents:
    Method: https://mirrors.tuna.tsinghua.edu.cn/debian/
    VerifyRelease: blindtrust
    FilterSrcList: purge ./mirror.packages.nonfree

conf/mirror.packages.main(可选示例):

    apt install
    dpkg    install
    base-files  install

### 仓库导入软件包

```shell
#导入deb软件包
$ reprepro -C ${Components} -A ${Architectures} -P ${priority} includedeb ${Codename} ${debian_package_filepath}

#导入dsc源码文件
$ reprepro -C ${Components} includedsc ${Codename} ${debian_source_filepath}
```

参数解释:
    
    -C ${Components}        #[可选]Components指定软件包导入到指定仓库组件，按照[仓库配置](#仓库配置)一节中的说明，Components可以为[main contrib non-free]中的某一个;
    -A ${Architectures}     #[可选]Architectures指定软件包导入的架构，按照[仓库配置](#仓库配置)一节中的说明，Architectures可以为[amd64 arm64]中的某一个;
    -P ${priority}          #[可选]priority指定软件包优先级，一般deb包打包错误时或者需要在includedeb阶段需要篡改软件包优先级时使用，可以设置[extra important optional required standard]中的某一个;

### 仓库删除软件包

```shell
#删除某一个deb软件包
$ reprepro remove ${Codename} ${package_name}

#以源码为单位,删除该源码目录下的所有deb以及源码
$ reprepro removesrc ${Codename} ${source_name}

#以源码为单位,删除多个源码目录下的所有deb以及源码
$ reprepro removesrcs ${Codename} ${source_nameA} ${source_nameB} ${source_nameC}
```

### 仓库更新

配置conf/distributions文件中Update字段等于conf/updates文件中的Name字段;

conf/updates文件中的FilterSrcList字段可选，也可以使用该字段进行管控当前仓库的软件包列表，可以实现仓库中的版本锁定、禁止未知软件进入仓库等；

```shell
#检查更新内容
$ reprepro dumpupdate ${codename}
$ reprepro checkupdate ${codename}

#仓库同步更新
$ reprepro update ${codename}
```

### 记一些reprepro其他操作

```shell
#列出仓库所有软件包
$ reprepro list ${codename}

#检查仓库deb与源码版本匹配报告
$ reprepro reportcruft ${codename}
```
