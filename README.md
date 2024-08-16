# Xensifter

## Requirements

- git
- build-essential
- docker

## Setup

### Xen

clone the repo and submodules
```shell
git submodule update --init --recursive
```

Install Xen with VM forking features
```shell
sudo apt-get update
sudo apt-get install -y iasl libyajl-dev libsystemd-dev ninja-build build-essential uuid-dev libncurses-dev pkg-config libglib2.0-dev libpixman-1-dev flex bison python3 python3-dev
cd xen
find ../patches/ -type f -name '*-xen-*' -exec git apply {} \;
echo CONFIG_EXPERT=y > xen/.config
echo CONFIG_MEM_SHARING=y >> xen/.config
make -C xen olddefconfig
./configure --enable-systemd --disable-docs --disable-stubdom --disable-pvshim --enable-githttp
make -j$(nproc) debball
sudo dpkg -i dist/xen-upstream-*.deb
sudo systemctl enable xencommons.service
sudo systemctl enable xen-qemu-dom0-disk-backend.service
sudo systemctl enable xen-init-dom0.service
sudo systemctl enable xenconsoled.service
echo "/usr/local/lib" | sudo tee -a /etc/ld.so.conf.d/xen.conf
echo "none /proc/xen xenfs defaults,nofail 0 0" | sudo tee -e /etc/fstab
ldconfig
make distclean
git clean -xdf
git reset --hard $(git describe --tags --abbrev=0)
```

Update Grub and adapt `dom0_mem` and `dom0_max_vcpus` settings as needed:
```shell
echo "GRUB_CMDLINE_XEN_DEFAULT=\"hap_1gb=false hap_2mb=false console=vga hpet=legacy-replacement dom0_mem=8096M dom0_max_vcpus=4 dom0_vcpus_pin=1 iommu=no-sharept spec-ctrl=0 xpti=0 vpmu=bts \\\"cpufreq=hwp:hdc=0;xen:performance,verbose\\\"\"" >> /etc/default/grub
sudo mv /etc/grub.d/20_linux_xen /etc/grub.d/09_linux_xen
sudo update-grub
```

### Xensifter

## Run

```shell
./run.sh
```

## Building the documentation

```shell
cd docs
make html
xdg-open build/html/index.html
```

## Disclaimer

Note: All components are provided for research and validation purposes only. Use at your own risk.
