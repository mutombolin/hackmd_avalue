# RK3568J (Debian 11)

# Build Debian 11

## References

- [**【電腦程式】Rockpi4 Image 編譯過程**](https://muta2001.pixnet.net/blog/post/54592636)

## Source

### From Compal

```bash
# .repo
rk356x_linux5.10_release_v1.4.0_20231220_rkr7.tar

# patch
rk3568debian110_patch.tar
```


### Gitlab

- clone all projects 

    ```bash
    git clone --recursive -j8 http://192.168.100.17/rockchip/rk3568j-debian11/rk3568debian110.git
    ```


## Docker

### Generate docker image

- Dockerfile
    
    ```
    # avalue/build-rk-debian-11:RK3568J
    FROM ubuntu:20.04

    #ENV DEBIAN_FRONTEND noninteractive # Not recommended
    ARG DEBIAN_FRONTEND=noninteractive

    ARG CUR_USER

    #RUN echo "nameserver 8.8.8.8" >> /etc/resolv.conf

    RUN apt-get update && apt-get -y upgrade

    # Build Host Packages
    # Installing required packages
    RUN apt-get install -y git ssh make gcc libssl-dev liblz4-tool expect \
    g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
    qemu-user-static live-build bison flex fakeroot cmake gcc-multilib \
    g++-multilib unzip device-tree-compiler ncurses-dev libgucharmap-2-90-dev \
    bzip2 expat gpgv2 cpp-aarch64-linux-gnu time mtd-utils \
    asciidoc \
    libgmp-dev libmpc-dev bc python-is-python3 python2 expect-dev rsync \
    bsdmainutils

    # Additional host packages required by network and others
    RUN apt-get install -y net-tools vim screen dialog

    # Create a non-root user that will perform the actual build
    RUN apt-get install -y sudo
    RUN echo "${CUR_USER} ALL=(ALL) NOPASSWD:ALL" | tee -a /etc/sudoers

    # Fix error "Please use a locale setting which supports utf-8."
    # See https://wiki.yoctoproject.org/wiki/TipsAndTricks/ResolvingLocaleIssues
    RUN apt-get install -y locales
    RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="en_US.UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8

    ENV LC_ALL en_US.UTF-8
    ENV LANG en_US.UTF-8
    ENV LANGUAGE en_US.UTF-8

    ENV USER=debian11-docker
    ARG USER_ID=0
    ARG GROUP_ID=0

    # Add a new user
    RUN groupadd -g ${GROUP_ID} jenkins-docker && useradd -m -g jenkins-docker -u ${USER_ID} debian11-docker && echo "debian11-docker:debian11-docker" | chpasswd && adduser debian11-docker sudo

    # extra packages
    COPY packages/* /packages/
    WORKDIR /packages
    RUN dpkg --force-all -i *
    RUN apt --fix-broken install -y

    # qemu-aarch64-static
    RUN update-binfmts --unimport qemu-aarch64 2>/dev/null
    RUN update-binfmts --disable qemu-aarch64 2>/dev/null
    RUN rm -rf /usr/bin/qemu-aarch64-static
    COPY bin/qemu-aarch64-static /usr/bin/
    RUN update-binfmts --enable qemu-aarch64 2>/dev/null
    RUN update-binfmts --import qemu-aarch64 2>/dev/null

    # repo
    COPY tools/repo /usr/bin/

    CMD "/bin/bash"
    WORKDIR /mnt/rk3568j
    USER debian11-docker
    # EOF
    ```

### Build Docker

- docker build
    
    ```bash
    sudo docker build -t rk3568j_debian11:1.0 --build-arg USER_ID=`id -u` --build-arg GROUP_ID=`id -g` --build-arg CUR_USER=debian11-docker -f /mnt/data2/projects/rk3568j_debian/docker/Dockerfile .
    ```

### Run Docker

- docker run
    
    ```bash
    sudo docker run -it -v /mnt/data2/projects/rk3568j_debian/source:/mnt/rk3568j --privileged=true rk3568j_debian11:1.0

    * privileged=true -- for mount loop device
    ```


## Build

- README.md
    
    ```bash
        # RK3568DEBIAN110

        #### Base on rk356x_linux5.10_release_v1.4.0_20231220_rkr7

        ### git clone repository
        ```sh
        $ git clone --branch master --single-branch git@gitlab.com:ipc-sdk/rk3568debian110.git
        ```

        ### How to use
        ```sh
        $ mkdir rk3568debian110_patch
        $ cd rk3568debian110_patch
        $ git clone git@gitlab.com:ipc-sdk/rk3568debian110.git
        $ cd ..
        $ mkdir rk3568debian110
        $ cd rk3568debian110
        $ tar xvf rk356x_linux5.10_release_v1.4.0_20231220_rkr7.tar
        $ repo sync -l
        $ cp -rf ../rk3568debian110_patch/rk3568debian110/* .
        $ ./delete.sh
        ```

        ### How to build debian OS
        ```sh
        $ cd debian
        $ RELEASE=bullseye TARGET=desktop ARCH=arm64 ./mk-base-debian.sh
        $ VERSION=debug ARCH=arm64 ./mk-rootfs-bullseye.sh
        $ ./mk-image.sh
        ```

        ### How to build recovery
        ```sh
        $ source buildroot/envsetup.sh (Select 57. rockchip_rk3568_recovery)
        $ ./build.sh recovery
        ```

        ### How to build difference panel
        ```sh
        10inch panel:
        export RK_ROOTFS_SYSTEM=debian
        export IPC_DISPLAY_SIZE=10
        ./build.sh rockchip_rk3568_evb1_ddr4_v10_defconfig
        ./build.sh

        7inch panel:
        export RK_ROOTFS_SYSTEM=debian
        export IPC_DISPLAY_SIZE=7
        ./build.sh rockchip_rk3568_evb1_ddr4_v10_for_7_lvds_defconfig
        ./build.sh
    ```


### repo

- Download repo from [https://storage.googleapis.com/git-repo-downloads/repo](https://storage.googleapis.com/git-repo-downloads/repo) 

    ```bash!
    $ curl https://storage.googleapis.com/git-repo-downloads/repo > repo
    $ chmod +x repo
    ```

### Preliminary
- command
    ```bash
    $ mkdir rk3568debian110_patch
    $ cd rk3568debian110_patch
    $ git clone git@gitlab.com:ipc-sdk/rk3568debian110.git
    $ cd ..
    $ mkdir rk3568debian110
    $ cd rk3568debian110
    $ tar xvf rk356x_linux5.10_release_v1.4.0_20231220_rkr7.tar
    $ **repo sync -l** 
    $ cp -rf ../rk3568debian110_patch/rk3568debian110/* .
    $ ./delete.sh
    ```

- repo sync -l
    ```
     -l, --local-only
    only update working tree, don’t fetch
    ```


### build debian os

```bash
$ cd debian
$ RELEASE=bullseye TARGET=desktop ARCH=arm64 ./mk-base-debian.sh
$ VERSION=debug ARCH=arm64 ./mk-rootfs-bullseye.sh
$ ./mk-image.sh
```

### build recovery

```bash
$ source buildroot/envsetup.sh (Select 57. rockchip_rk3568_recovery)
$ ./build.sh recovery
```

### build 7” & 10” panel

- 10 inch panel
    
    ```bash
    export RK_ROOTFS_SYSTEM=debian
    export IPC_DISPLAY_SIZE=10
    ./build.sh rockchip_rk3568_evb1_ddr4_v10_defconfig
    ./build.sh
    ```
    
- 7 inch panel
    
    ```bash
    export RK_ROOTFS_SYSTEM=debian
    export IPC_DISPLAY_SIZE=7
    ./build.sh rockchip_rk3568_evb1_ddr4_v10_for_7_lvds_defconfig
    ./build.sh
    ```
    

### Log Files

- build logs
    
    ```bash
    output/log
    ```
    

## U-Boot

- U-Boot version (**2017.09-g2a03f1a660-231011**)
    - From u-boot.bin file (u-boot.bin)
        
        ```bash
        mutombo@Z790-AVALUE:/mnt/data2/projects/rk3568j_debian/source/tmp/rk3568debian110/u-boot$ strings u-boot.bin | grep "U-Boot" | tail -n1
        U-Boot 2017.09-g2a03f1a660-231011 #debian11-docker (Mar 14 2025 - 07:22:19 +0000)
        ```
        
    - From `/proc/cmdline`
        
        ```bash
        root@linaro-alip:/# cat /proc/cmdline
        storagemedia=emmc androidboot.storagemedia=emmc androidboot.mode=normal  androidboot.sku=10 androidboot.verifiedbootstate=orange rw rootwait earlycon=u
        art8250,mmio32,0xfe660000 console=ttyFIQ0 root=PARTUUID=614e0000-0000 androidboot.fwver=spl-v1.13,bl31-v1.44,uboot-2a03f1a660-03/14/2025
        ```
        
    - From the device (/dev/mmcblk0p1)
        
        ```bash
        root@linaro-alip:/# strings /dev/mmcblk0p1 | grep "U-Boot" | tail -n1
        U-Boot 2017.09-g2a03f1a660-231011 #debian11-docker (Mar 14 2025 - 06:50:55 +0000)
        ```
        

## Rockchip SDK

- sdk version (**1.4.0_20231220**)
    
    from README.md file
    

## Buildroot

- buildroot version (**2021.11**)
    
    ```bash
    mutombo@Z790-AVALUE:/mnt/data2/projects/rk3568j_debian/source/tmp/rk3568debian110/buildroot$ cat CHANGES | head -n1
    2021.11, released December 5th, 2021
    ```
    

## Kernel

- kernel version (**5.10.198**)
    
    ```bash
    debian11-docker@5124ac27b3ec:/mnt/rk3568j/tmp/rk3568debian110/kernel$ make kernelversion
    5.10.198
    ```
    
- kernel version from boot.img
    - strings boot.img | grep ‘Linux version’
        
        ```bash
        debian11-docker@5148f48dfc6a:/mnt/rk3568j/tmp/rk3568debian110/kernel$ strings boot.img | grep 'Linux version'
        Linux version 5.10.198 (debian11-docker@5148f48dfc6a) (aarch64-none-linux-gnu-gcc (GNU Toolchain for the A-profile Architecture 10.3-2021.07 (arm-10.29)) 10.3.1 20210621, GNU ld (GNU Toolchain for the A-profile Architecture 10.3-2021.07 (arm-10.29)) 2.36.1.20210621) #14 SMP Mon Mar 17 02:50:57 UTC 2025
        ```
        

## Yocto

- Yocto related information
    - References
        - [How to find out Yocto version](https://stackoverflow.com/questions/32180314/how-to-find-out-yocto-version)
    - DISTRO_REL_TAG (**4.0**)
        
        ```bash
        mutombo@Z790-AVALUE:/mnt/data2/projects/rk3568j_debian/source/tmp/rk3568debian110/yocto$ cat poky/documentation/poky.yaml.in | grep DISTRO_REL_TAG
        DISTRO_REL_TAG : "yocto-4.0"
        ```
        
    - DISTRO info
        
        ```bash
        mutombo@Z790-AVALUE:/mnt/data2/projects/rk3568j_debian/source/tmp/rk3568debian110/yocto$ cat poky/documentation/poky.yaml.in | grep DISTRO
        DISTRO : "4.0"
        DISTRO_NAME_NO_CAP : "kirkstone"
        DISTRO_NAME : "Kirkstone"
        DISTRO_NAME_NO_CAP_MINUS_ONE : "honister"
        DISTRO_NAME_NO_CAP_LTS : "dunfell"
        DISTRO_REL_TAG : "yocto-4.0"
        YOCTO_RELEASE_DL_URL : "&YOCTO_DL_URL;/releases/yocto/yocto-&DISTRO;"
        ```
        
    - YOCTO info
        
        ```bash
        mutombo@Z790-AVALUE:/mnt/data2/projects/rk3568j_debian/source/tmp/rk3568debian110/yocto$ cat poky/documentation/poky.yaml.in | grep "YOCTO*"
        YOCTO_DOC_VERSION : "4.0"
        YOCTO_DL_URL : "https://downloads.yoctoproject.org"
        YOCTO_AB_URL : "https://autobuilder.yoctoproject.org"
        YOCTO_RELEASE_DL_URL : "&YOCTO_DL_URL;/releases/yocto/yocto-&DISTRO;"
        ```
        

## IO

### CAN

- Add device can0 & can1 to rk3568-evb.dtsi

    - arch/arm64/boot/dts/rockchip/rk3568-evb.dtsi
    
        ```bash
        diff --git a/arch/arm64/boot/dts/rockchip/rk3568-evb.dtsi b/arch/arm64/boot/dts/rockchip/rk3568-evb.dtsi
        index 8357f7e97346..538982ea54ad 100644
        --- a/arch/arm64/boot/dts/rockchip/rk3568-evb.dtsi
        +++ b/arch/arm64/boot/dts/rockchip/rk3568-evb.dtsi
        @@ -370,6 +370,30 @@ &bus_npu {
                status = "okay";
         };

        +&can0 {
        +        assigned-clocks = <&cru CLK_CAN0>;
        +        assigned-clock-rates = <150000000>;
        +        pinctrl-names = "default";
        +        pinctrl-0 = <&can0m1_pins>;
        +        status = "disabled";
        +};
        +
        +&can1 {
        +        assigned-clocks = <&cru CLK_CAN1>;
        +        assigned-clock-rates = <150000000>;
        +        pinctrl-names = "default";
        +        pinctrl-0 = <&can1m1_pins>;
        +        status = "disabled";
        +};
        +
        +&can2 {
        +        assigned-clocks = <&cru CLK_CAN2>;
        +        assigned-clock-rates = <150000000>;
        +        pinctrl-names = "default";
        +        pinctrl-0 = <&can2m0_pins>;
        +        status = "okay";
        +};
        +
         &cpu0 {
                cpu-supply = <&vdd_cpu>;
         };
        ```
    

## Image

```
/output/update/Image
```
## Errors

### Syntax Error: invalid syntax to repo init in the AOSP code

- Workaround
    - Download the latest repo
            
            ```
            curl https://storage.googleapis.com/git-repo-downloads/repo-1 > ~/bin/repo
            chmod a+x ~/bin/repo
            python3 ~/bin/repo init -u git@....
            ```
            
### ModuleNotFoundError: No module named ‘formatter’
- Reason
        
    ```
        ubantu22.04来说formatter已在python3.4+标记成废弃接口，就算你按照网上教程添加这个模块也无法解决。
    ```
        
- Workaround
    - Use ubuntu 20.04
- References
    - [**ModuleNotFoundError: No module named 'formatter'**](https://www.cnblogs.com/Sandals-little/p/18134137)

### cannot stat 'chroot.files': No such file or directory
- Error Messages
        
    ```bash
    P: Running debootstrap second stage under QEMU
    W: Failure trying to run:  /sbin/ldconfig
    W: See //debootstrap/debootstrap.log for details
    E: An unexpected failure occurred, exiting...
    tar -jcf linaro-bullseye-alip-`date +%Y%m%d`-1.config.tar.bz2 auto/ config/ configure;
    sudo mv chroot.files linaro-bullseye-alip-`date +%Y%m%d`-1.contents;
    mv: cannot stat 'chroot.files': No such file or directory
    Makefile:22: recipe for target 'all' failed
    make: *** [all] Error 1
    ERROR: Running build_debian failed!
    ERROR: exit code 2 from line 1405:
        RELEASE=$RK_DEBIAN_VERSION TARGET=desktop ARCH=$ARCH ./mk-base-debian.sh
    ```
        
- Reason
        
    ```bash
    因為使用的是 Ubuntu 18.0.4 的版本，qemu 最高版本為 2.11.1, 導致版本過低所以編譯不過
    ```
        
- Workaround
        
    ```bash
    1. 把系統版本升級為 20.0.4 版本即可，這樣 qemu 的版本為 4.2.1，即可編譯通過
    2. 安裝 "qemu_5.2+dfsg-11_amd64.deb" (ubuntu-build-service/packages)
    ```
        
- References
    - [**rockchip Debian 编译报错**](https://blog.csdn.net/badbayyj/article/details/129210477)
### lb config: unrecognized option ‘--debootstrap-options‘
- Error Messages
        
    ```bash
    lb config: unrecognized option ‘--debootstrap-options‘
    ```
        
- Reason
        
    Missing packages
        
- Workaround
        
    Install the missing packages at ubuntu-build-service/packages 
        
    ```bash
    sudo dpkg -i ubuntu-build-service/packages/*
    ```
        
- References
    - [rk3399-linux_v2.8 编译报错 lb config: unrecognized option ‘--debootstrap-options‘](https://blog.csdn.net/jiangdou88/article/details/127766118)
    
### qemu-aarch64 too old (4.2.1)
- workaround
    ```bash
    sudo update-binfmts --unimport qemu-aarch64 2>/dev/null
    sudo update-binfmts --disable qemu-aarch64 2>/dev/null
    sudo rm -f /usr/bin/qemu-aarch64-static
    sudo cp /mnt/rk3568j/rk3568debian110/device/rockchip/common/data/qemu-aarch64-static /usr/bin/
    sudo update-binfmts --enable qemu-aarch64 2>/dev/null
    sudo update-binfmts --import qemu-aarch64 2>/dev/null
    ```
    

### disk space error
- Error
        
    ```
    apt-get update
    Get:1 http://debian.csie.ntu.edu.tw/debian bullseye InRelease [116 kB]
    Err:1 http://debian.csie.ntu.edu.tw/debian bullseye InRelease
      **Error writing to file - write (28: No space left on device)** [IP: 140.112.30.75 80]
    ```
        
    debian 11 from source
    ![image](https://hackmd.io/_uploads/HJraBDHn1e.png)

    debian 11 from Compal
    ![image 1](https://hackmd.io/_uploads/r1-iSwHh1g.png)
        
- Evaluation
    - Use the df command to get the disk usage information as shown below. The root file system has run out of space.
            
        ```
        root@linaro-alip:/var/log# df -h
        Filesystem      Size  Used Avail Use% Mounted on
        /dev/root       3.0G  2.9G     0 100% /
        devtmpfs        1.9G  8.0K  1.9G   1% /dev
        tmpfs           2.0G     0  2.0G   0% /dev/shm
        tmpfs           781M  1.5M  780M   1% /run
        tmpfs           5.0M  4.0K  5.0M   1% /run/lock
        tmpfs           391M   28K  391M   1% /run/user/1000
        tmpfs           391M   28K  391M   1% /run/user/110
        ```
            
    - Reason
        
        The files `“resize-helper”` and `“disk-helper”` under “debian/overlay/usr/bin” are different from Compal’s prebuilt version. 
        
    - Workaround
        - Replace the current files with Compal’s prebuilt ones.

### DNS problem with LTE (Quectel EM060K)
- `/etc/resolv.conf` should be a symlink to `/run/resolvconf/resolv.conf` ; otherwise, it won’t be updated after the LTE link (`ppp0`) is established.
- Update the resolv.conf file by the using `/usr/sbin/resolvconf`  command.
        
    ```
    $ sudo resolvconf -u
    ```
        

## Misc

### OTG

- Enable USB OTG, switch jump to 1-2 or remove the jump mounted on the JUSBID
    
    ![image 2](https://hackmd.io/_uploads/By3zLPrhye.png)

    

### Default

- username/password
    - linaro/linaro
- kernel config
    - rockchip_linux_defconfig
    - kernel/arch/arm64/boot/dts/rockchip/rk3568-evb1-ddr4-v10-linux.dts