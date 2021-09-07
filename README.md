# 在Amazon c6g实例上编译安装Greenplum并制作RPM安装包

## 编译安装最新版Greenplum

**在AWS管理控制台创建c6g实例，操作系统为Amazon Linux 2，ssh登录该操作系统，执行如下脚本：**


```
sudo yum -y update
sudo yum -y upgrade
sudo yum -y git
sudo amazon-linux-extras install epel
git clone https://github.com/greenplum-db/gpdb.git
cd gpdb
sh README.CentOS.bash
./configure --with-perl --with-python --with-libxml --with-gssapi --prefix=/usr/local/gpdb
make -j $(nproc)
sudo make install

```



## 制作Greenplum的RPM安装包

**安装rpm-build**

```
$ sudo yum install rpm-build
```

**在/root目录下建rpmbuild目录**

```
$ sudo su
$ mkdir -p ~/rpmbuild/BUILD ~/rpmbuild/RPMS ~/rpmbuild/BUILDROOT ~/rpmbuild/SRPMS ~/rpmbuild/SOURCES ~/rpmbuild/SPECS
```

**编辑greenplum6.16.3.spec**

```
$ vi ~/rpmbuild/SPECS/greenplum6.16.3.spec
```

```
%define version 6.16.3
%define directory /usr/local
%define __os_install_post %{nil}
%define debug_package %{nil}
%define __jar_repack 0


Name: greenplum
Version: %{version}
Release: 1%{?dist}
Summary: Greenplum Database
License: FIXME

# Url:
# Group
Source: %{name}-%{version}.tar.gz
# Patch:
# BuildRequires:
Requires: epel-release apr-devel bison bzip2-devel cmake3 flex gcc gcc-c++ krb5-devel libcurl-devel
Requires: libevent-devel libkadm5 libyaml-devel libxml2-devel libzstd-devel openssl-devel perl-ExtUtils-Embed python3-devel
Requires: python3-pip readline-devel xerces-c-devel zlib-devel postgresql postgresql-devel
# PreReq:
# Providers:
BuildRoot: %{_tmppath}/%{name}-%{version}-build


%description
This is GPDB RPM package, which does nothing.
%prep
%setup -q
%build
./configure --with-perl --with-python --with-libxml —with-gssapi —prefix=%{directory}/%{name}-%{version}
make %{?_smp_mflags}

%install
make install DESTDIR=%{buildroot} %{?_smp_mflags}
# cp -r %{_builddir}/%{name}-%{version}/ext %{buildroot}/%{directory}/%{name}-%{version}
%pre
%preun
%post
%postun
%clean
rm -rf %{buildroot}
%files
%defattr(-,root,root)
%{directory}/%{name}-%{version}
%changelog
```


**修改greenplum源代码目录名为greenplum-6.16.3并压缩：**

```
$ sudo su
$ mv gpdb greenplum-6.16.3
$ tar czvf greenplum-6.16.3.tar.gz greenplum-6.16.3
```

**把greenplum源码包复制到rpmbuild源代码目录**

```
$ cp greenplum-6.16.3.tar.gz ~/rpmbuild/SOURCES/
```

**制作rpm安装包：**

```
$ rpmbuild -ba ~/rpmbuild/SPECS/greenplum6.16.3.spec
```

**查看制作好的rpm安装包：**

```
$ ll ~/rpmbuild/RPMS/aarch64/
total 10964
-rw-r--r-- 1 root root 11226320 Jul  3 14:03 greenplum-6.16.3-1.amzn2.aarch64.rpm
```



## 在一台新的示例上验证Greenplum RPM安装包

**安装依赖包，执行如下脚本：**

```
sudo yum -y update
sudo yum -y upgrade
sudo yum -y git
sudo amazon-linux-extras install epel
sudo yum install -y epel-release
sudo yum install -y \
    apr-devel \
    bison \
    bzip2-devel \
    cmake3 \
    flex \
    gcc \
    gcc-c++ \
    krb5-devel \
    libcurl-devel \
    libevent-devel \
    libkadm5 \
    libyaml-devel \
    libxml2-devel \
    libzstd-devel \
    openssl-devel \
    perl-ExtUtils-Embed \
    python3-devel \
    python3-pip \
    readline-devel \
    xerces-c-devel \
    zlib-devel

sudo yum install -y \
    postgresql \
    postgresql-devel

sudo pip3 install conan
sudo pip3 install psutil==5.7.0
sudo pip3 install pygresql==5.2
sudo pip3 install pyyaml==5.3.1

```

**安装制作好的greenplum rpm包：**

```
$ sudo yum install greenplum-6.16.3-1.amzn2.aarch64.rpm

$ ls /usr/local/greenplum-6.16.3/
bin  docs  greenplum_path.sh  include  lib  libexec  sbin  share
```

