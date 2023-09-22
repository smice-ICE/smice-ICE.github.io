# How to add new provider driver in rdma-core

 ## 1. Add new provider and build

### 1.1.rdma-core source tree

```shell
rdma-core
├── buildlib
├── build.sh
├── ccan
├── CMakeLists.txt
├── COPYING.BSD_FB
├── COPYING.BSD_MIT
├── COPYING.GPL2
├── COPYING.md
├── debian
├── Documentation
├── ibacm
├── infiniband-diags
├── iwpmd
├── kernel-boot
├── kernel-headers
├── libibmad
├── libibnetdisc
├── libibumad
├── libibverbs
├── librdmacm
├── MAINTAINERS
├── providers
│   ├── bnxt_re
│   ├── cxgb4
│   ├── efa
│   ├── erdma
│   ├── hfi1verbs
│   ├── hns
│   ├── ipathverbs
│   ├── irdma
│   ├── mana
│   ├── mlx4
│   ├── mlx5
│   ├── mthca
│   ├── ocrdma
│   ├── qedr
|   ├── rib
│   ├── rxe
│   ├── siw
│   └── vmw_pvrdma
├── pyverbs
├── rdma-ndd
├── README.md
├── redhat
├── srp_daemon
├── suse
├── tests
└── util
```

- 其中providers是各个厂商的用户态驱动

  - **`rib`是resnics的用户态驱动,把驱动代码拷贝到`providers`下**

- **头文件`rib-abi.h`需要拷贝到`kernel-headers/rdma/`下**

- **kernel-headers/rdma/ib_user_ioctl_verbs.h中**

  ```diff
  @@ -254,6 +254,7 @@ enum rdma_driver_id {
          RDMA_DRIVER_SIW,
          RDMA_DRIVER_ERDMA,
          RDMA_DRIVER_MANA,
  +       RDMA_DRIVER_RIB,
   };
  ```

- **kernel-headers/CMakeLists.txt**

  ```diff
  @@ -26,6 +26,7 @@ publish_internal_headers(rdma
     rdma/rvt-abi.h
     rdma/siw-abi.h
     rdma/vmw_pvrdma-abi.h
  +  rdma/rib-abi.h
     )
  
   publish_internal_headers(rdma/hfi
  @@ -80,6 +81,7 @@ rdma_kernel_provider_abi(
     rdma/rdma_user_rxe.h
     rdma/siw-abi.h
     rdma/vmw_pvrdma-abi.h
  +  rdma/rib-abi.h
     )
  ```

- **CMakeLists.txt中添加rib子目录**

  ```diff
  @@ -733,6 +733,7 @@ add_subdirectory(providers/mthca)
   add_subdirectory(providers/ocrdma)
   add_subdirectory(providers/qedr)
   add_subdirectory(providers/vmw_pvrdma)
  +add_subdirectory(providers/rib)
   endif()
  ```

### 1.2.build

1. 按照`README.md`中的指导安装依赖包
2. 执行`bash build.sh`进行编译即可

### 1.3.patch制作(后续rpm包制作时需要)

1. 查看commit id, 然后记下commit id 前8位或者全部

   ```shell
   git log -n 1
   ```

2. 提交当前修改



## rdma-core rib rpm包制作

### libibverbs.rpm librdmacm.rpm rdma-core.rpm rdma-core-devel.rpm包结构和文件分析

- 这些rpm包都是公版发行包
- rdma-core-devel是用于rdma用户态APP开发的
- 还有未列出的**`libibverbs-utils.rpm`**和**`librdmacm-utils.rpm`**这两个包，里面是关于rdma的一些工具和示例可执行程序

```shell
.
├── libibverbs
│ ├── etc
│ │ └── libibverbs.d
│ │     ├── bnxt_re.driver
│ │     ├── cxgb4.driver
│ │     ├── efa.driver
│ │     ├── hfi1verbs.driver
│ │     ├── hns.driver
│ │     ├── i40iw.driver
│ │     ├── mlx4.driver
│ │     ├── mlx5.driver
│ │     ├── qedr.driver
│ │     ├── rxe.driver
│ │     ├── siw.driver
│ │     └── vmw_pvrdma.driver
│ └── usr
│     ├── lib
│     ├── lib64
│     │ ├── libefa.so.1 -> libefa.so.1.1.32.0
│     │ ├── libefa.so.1.1.32.0
│     │ ├── libibverbs
│     │ │ ├── libbnxt_re-rdmav25.so
│     │ │ ├── libcxgb4-rdmav25.so
│     │ │ ├── libefa-rdmav25.so -> ../libefa.so.1.1.32.0
│     │ │ ├── libhfi1verbs-rdmav25.so
│     │ │ ├── libhns-rdmav25.so
│     │ │ ├── libi40iw-rdmav25.so
│     │ │ ├── libmlx4-rdmav25.so -> ../libmlx4.so.1.0.32.0
│     │ │ ├── libmlx5-rdmav25.so -> ../libmlx5.so.1.16.32.0
│     │ │ ├── libqedr-rdmav25.so
│     │ │ ├── librxe-rdmav25.so
│     │ │ ├── libsiw-rdmav25.so
│     │ │ └── libvmw_pvrdma-rdmav25.so
│     │ ├── libibverbs.so.1 -> libibverbs.so.1.11.32.0
│     │ ├── libibverbs.so.1.11.32.0
│     │ ├── libmlx4.so.1 -> libmlx4.so.1.0.32.0
│     │ ├── libmlx4.so.1.0.32.0
│     │ ├── libmlx5.so.1 -> libmlx5.so.1.16.32.0
│     │ └── libmlx5.so.1.16.32.0
│     └── share
│         ├── doc
│         │ └── rdma-core
│         │     ├── libibverbs.md
│         │     ├── rxe.md
│         │     └── tag_matching.md
│         └── man
│             └── man7
│                 ├── mlx4dv.7.gz
│                 ├── mlx5dv.7.gz
│                 └── rxe.7.gz
├── libibverbs-32.0-4.el8.x86_64.rpm
├── librdmacm
│ └── usr
│     ├── lib
│     ├── lib64
│     │ ├── librdmacm.so.1 -> librdmacm.so.1.3.32.0
│     │ ├── librdmacm.so.1.3.32.0
│     │ └── rsocket
│     │     ├── librspreload.so
│     │     ├── librspreload.so.1 -> librspreload.so
│     │     └── librspreload.so.1.0.0 -> librspreload.so
│     └── share
│         ├── doc
│         │ └── rdma-core
│         │     └── librdmacm.md
│         └── man
│             └── man7
│                 └── rsocket.7.gz
├── rdma-core
│ ├── etc
│ │ ├── modprobe.d
│ │ │ ├── mlx4.conf
│ │ │ └── truescale.conf
│ │ ├── rdma
│ │ │ ├── mlx4.conf
│ │ │ └── modules
│ │ │     ├── infiniband.conf
│ │ │     ├── iwarp.conf
│ │ │     ├── opa.conf
│ │ │     ├── rdma.conf
│ │ │     └── roce.conf
│ │ └── udev
│ │     └── rules.d
│ │         └── 70-persistent-ipoib.rules
│ └── usr
│     ├── bin
│     │ └── ibdev2netdev
│     ├── lib
│     │ ├── dracut
│     │ │ └── modules.d
│     │ │     └── 05rdma
│     │ │         └── module-setup.sh
│     │ ├── modprobe.d
│     │ │ └── libmlx4.conf
│     │ ├── systemd
│     │ │ └── system
│     │ │     ├── rdma-hw.target
│     │ │     ├── rdma-load-modules@.service
│     │ │     └── rdma-ndd.service
│     │ └── udev
│     │     ├── rdma_rename
│     │     └── rules.d
│     │         ├── 60-rdma-ndd.rules
│     │         ├── 60-rdma-persistent-naming.rules
│     │         ├── 75-rdma-description.rules
│     │         ├── 90-rdma-hw-modules.rules
│     │         ├── 90-rdma-ulp-modules.rules
│     │         └── 90-rdma-umad.rules
│     ├── libexec
│     │ ├── mlx4-setup.sh
│     │ └── truescale-serdes.cmds
│     ├── sbin
│     │ └── rdma-ndd
│     └── share
│         ├── doc
│         │ └── rdma-core
│         │     ├── README.md
│         │     └── udev.md
│         ├── licenses
│         │ └── rdma-core
│         │     ├── COPYING.BSD_FB
│         │     ├── COPYING.BSD_MIT
│         │     ├── COPYING.GPL2
│         │     └── COPYING.md
│         └── man
│             ├── man7
│             │ └── rxe.7.gz
│             └── man8
│                 └── rdma-ndd.8.gz
├── rdma-core-devel
│ └── usr
│     ├── include
│     │ ├── infiniband
│     │ │ ├── acm.h
│     │ │ ├── acm_prov.h
│     │ │ ├── arch.h
│     │ │ ├── efadv.h
│     │ │ ├── ib.h
│     │ │ ├── ibnetdisc.h
│     │ │ ├── ibnetdisc_osd.h
│     │ │ ├── ib_user_ioctl_verbs.h
│     │ │ ├── mad.h
│     │ │ ├── mad_osd.h
│     │ │ ├── mlx4dv.h
│     │ │ ├── mlx5_api.h
│     │ │ ├── mlx5dv.h
│     │ │ ├── mlx5_user_ioctl_verbs.h
│     │ │ ├── opcode.h
│     │ │ ├── sa.h
│     │ │ ├── sa-kern-abi.h
│     │ │ ├── tm_types.h
│     │ │ ├── umad_cm.h
│     │ │ ├── umad.h
│     │ │ ├── umad_sa.h
│     │ │ ├── umad_sa_mcm.h
│     │ │ ├── umad_sm.h
│     │ │ ├── umad_str.h
│     │ │ ├── umad_types.h
│     │ │ ├── verbs_api.h
│     │ │ └── verbs.h
│     │ └── rdma
│     │     ├── rdma_cma_abi.h
│     │     ├── rdma_cma.h
│     │     ├── rdma_verbs.h
│     │     └── rsocket.h
│     ├── lib64
│     │ ├── libefa.so -> libefa.so.1
│     │ ├── libibmad.so -> libibmad.so.5
│     │ ├── libibnetdisc.so -> libibnetdisc.so.5
│     │ ├── libibumad.so -> libibumad.so.3
│     │ ├── libibverbs.so -> libibverbs.so.1
│     │ ├── libmlx4.so -> libmlx4.so.1
│     │ ├── libmlx5.so -> libmlx5.so.1
│     │ ├── librdmacm.so -> librdmacm.so.1
│     │ └── pkgconfig
│     │     ├── libefa.pc
│     │     ├── libibmad.pc
│     │     ├── libibnetdisc.pc
│     │     ├── libibumad.pc
│     │     ├── libibverbs.pc
│     │     ├── libmlx4.pc
│     │     ├── libmlx5.pc
│     │     └── librdmacm.pc
│     └── share
│         ├── doc
│         │ └── rdma-core
│         │     └── MAINTAINERS
│         └── man
│             ├── man3
│             │ ├── efadv_create_driver_qp.3.gz
│             │ ├── ...
│             │ └── umad_unregister.3.gz
│             └── man7
│                 ├── efadv.7.gz
│                 ├── mlx4dv.7.gz
│                 ├── mlx5dv.7.gz
│                 └── rdma_cm.7.gz
```

### rdma-core-provider-rib.rpm 制作(以Centos为例)

1. rpm环境初始化

   1. 安装rpm-build: `sudo yum install rpm-build`
   2. 创建rpm工作目录: `mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}`
   3. 初始化rpmbuild的环境配置：`echo '%_topdir %(echo $HOME)/rpmbuild' > ~/.rpmmacros`

2. 下载目标rdma-core公版源码到`rpmbuild/SOURCES`中，这个源码文件和`rdma-core.spec`中要求的一致

   ```shell
   rdma-core-42.0.tgz
   ```

3. 拷贝provider driver patch包到`rpmbuild/SOURCES`中，patch的生成方式参考build的1.1和1.3

   ```shell
   rdma-core-rib.patch
   ```

4. 下载目标`rdma-core/redhat/rdma-core.spec`到`rpmbuild/SPEC`中

   - 修改以下几行
     - 声明patch包,需要在Source之后buildrequires之前`Patch0: rdma-core-rib.patch`需要和`SOURCES`中的patch报名一致

   ```diff
   @@ -10,6 +10,9 @@ Summary: RDMA core userspace libraries and daemons
    License: GPLv2 or BSD
    Url: https://github.com/linux-rdma/rdma-core
    Source: rdma-core-%{version}.tar.gz
   +
   +Patch0: rdma-core-rib.patch
   +
    # Do not build static libs by default.
    %define with_static %{?_with_static: 1} %{?!_with_static: 0}
   
   @@ -174,6 +177,8 @@ Provides: libocrdma = %{version}-%{release}
    Obsoletes: libocrdma < %{version}-%{release}
    Provides: librxe = %{version}-%{release}
    Obsoletes: librxe < %{version}-%{release}
   +Provides: librib = %{version}-%{release}
   +Obsoletes: librib < %{version}-%{release}
   
    %description -n libibverbs
    libibverbs is a library that allows userspace processes to use RDMA
   @@ -197,6 +202,7 @@ Device-specific plug-in ibverbs userspace drivers are included:
    - libmthca: Mellanox InfiniBand HCA
    - libocrdma: Emulex OneConnect RDMA/RoCE Device
    - libqedr: QLogic QL4xxx RoCE HCA
   +- librib: A software implementation of the RoCE protocol
    - librxe: A software implementation of the RoCE protocol
    - libsiw: A software implementation of the iWarp protocol
    - libvmw_pvrdma: VMware paravirtual RDMA device
   @@ -282,6 +288,8 @@ easy, object-oriented access to IB verbs.
    %prep
    %setup
   
   +%patch0 -p1
   +
    %build
   ```

5. 然后再`SPECS`下执行命令

   ```shell
   rpmbuild -ba rdma-core.spec
   ```

6. 命令过程如果提示依赖包缺失，根据提示安装相应依赖包即可，然后等待执行完成

   - 以下就是执行完命令的目录结构

     ```shell
     ./rpmbuild/
     ├── BUILD
     │   └── rdma-core-48.0
     │       ├── bin
     ...
     │       └── util
     ├── BUILDROOT
     ├── RPMS
     │   └── x86_64
     │       ├── ibacm-48.0-1.el8.x86_64.rpm
     │       ├── ibacm-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── infiniband-diags-48.0-1.el8.x86_64.rpm
     │       ├── infiniband-diags-compat-48.0-1.el8.x86_64.rpm
     │       ├── infiniband-diags-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── iwpmd-48.0-1.el8.x86_64.rpm
     │       ├── iwpmd-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── libibumad-48.0-1.el8.x86_64.rpm
     │       ├── libibumad-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── libibverbs-48.0-1.el8.x86_64.rpm
     │       ├── libibverbs-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── libibverbs-utils-48.0-1.el8.x86_64.rpm
     │       ├── libibverbs-utils-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── librdmacm-48.0-1.el8.x86_64.rpm
     │       ├── librdmacm-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── librdmacm-utils-48.0-1.el8.x86_64.rpm
     │       ├── librdmacm-utils-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── python3-pyverbs-48.0-1.el8.x86_64.rpm
     │       ├── python3-pyverbs-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── rdma-core-48.0-1.el8.x86_64.rpm
     │       ├── rdma-core-debuginfo-48.0-1.el8.x86_64.rpm
     │       ├── rdma-core-debugsource-48.0-1.el8.x86_64.rpm
     │       ├── rdma-core-devel-48.0-1.el8.x86_64.rpm
     │       ├── srp_daemon-48.0-1.el8.x86_64.rpm
     │       └── srp_daemon-debuginfo-48.0-1.el8.x86_64.rpm
     ├── SOURCES
     │   ├── rdma-core-48.0.tar.gz
     │   └── rdma-core-rib.patch
     ├── SPECS
     │   └── rdma-core.spec
     └── SRPMS
         └── rdma-core-48.0-1.el8.src.rpm
     ```

   - BUILD是打包过程中文件

   - RPMS/x86_64/下面就是rdma-core打包好的各个rpm包

   - SRPMS中的rdma-core.src.rpm中包含的是制作rpm的原文件

     ```shell
     ├── rdma-core-48.0.tar.gz
     ├── rdma-core-rib.patch
     └── rdma-core.spec
     ```

   

   

   

