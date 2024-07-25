# POWERSHELL COMMANDS

```powershell
# --list --verbose
wsl -l -v
# --terminate
wsl -t <distro name>
# -- upgrade
wsl --set-version <distro name> 2 
# --unregister
wsl --unregister <distro name>
```

# ENABLE WSL BY POWERSHELL COMMAND

```powershell
# enable virtual machine platform
Get-WindowsOptionalFeature -Online | findstr /I 'virtual'
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform

# enable wsl
Get-WindowsOptionalFeature -Online | findstr /I 'subsystem'
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

## UPGRADE TO WSL2

> https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

```powershell
wsl --set-default-version 2
wsl --set-version <distro name> 2 
```

## CONFIGURE

https://learn.microsoft.com/en-us/windows/wsl/wsl-config

# INSTALL OS

## UBUNTU 2204

https://learn.microsoft.com/en-us/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package

```powershell
Invoke-WebRequest -Uri https://aka.ms/wslubuntu2204 -OutFile Ubuntu2204.appx -UseBasicParsing
Add-AppxPackage .\Ubuntu2204.appx
double-click Ubuntu2204.appx
```

# WINDOWS TERMINAL

```powershell
winget install Microsoft.WindowsTerminal
```

# UBUNTU IN 2204

## VISIT FILESYSTEM VIA WINDOWS

```sh
\\wsl$
```

# WSL INIT

## CHANGE APT SOURCE

### TSINGHUA

```sh
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

### ALIYUN

```sh
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo apt update
```

## SET UP ZSH

```sh
sudo apt install -y zsh git
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"


sed -i 's/ZSH_THEME=.*/ZSH_THEME="itchy"/' ~/.zshrc
tee -a ~/.zshrc << EOF
export VISUAL="vim"
EOF
source ~/.zshrc

# SET UP GIT
git config --global credential.helper store
git config --global core.quotepath false
git config --global oh-my-zsh.hide-dirty 1

# SET UP WORKING PATH
ln -s /mnt/c/Users/xxx/Raymond/Downloads downloads
ln -s /mnt/c/Users/xxx/Raymond/Workspace workspace
```

## FONT & INPUT METHOD

```sh
sudo apt install -y fontconfig language-pack-zh-hans
sudo dpkg-reconfigure locales
- check en_us.utf-8 and zh_cn.utf-8
- set default: en_us.uft-8

sudo ln -s /mnt/c/Windows/Fonts /usr/share/fonts/truetype/windows
fc-cache -fv

sudo apt install fcitx dbus-x11 im-config  fcitx-table-wubi

tee -a ~/.profile << EOF
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export DefaultIMModule=fcitx
fcitx-autostart &>/dev/null
EOF

source ~/.profile
fcitx-configtool
```

## TRUST ROOT CA

```sh
# solve curl & wget cert issue
openssl s_client -showcerts -verify 5 -connect www.baidu.com:443
> save root ca as zscaler_root_ca.crt
sudo cp zscaler_root_ca.crt /usr/local/share/ca-certificates
sudo update-ca-certificates

# solve java application cert issue
cd $JAVA_HOME/lib/security
# cd /usr/share/dbeaver-ce/jre/lib/security
sudo keytool -import -alias zscaler1 -keystore cacerts -file /usr/local/share/ca-certificates/zscaler_root_ca.crt -trustcacerts
> password: changeit
> Trust this certificate? [no]:  yes

# sole vscode cert issue
code --ignore-certificate-errors
```


## SYSTEMD & DOMAIN RESOLVE

```sh
sudo tee /etc/wsl.conf << EOF
[boot]
systemd=true

[network]
generateHosts = false
generateResolvConf = false
EOF

sudo vim /etc/systemd/resolved.conf

sudo systemctl restart systemd-resolved
```

```ini
[Resolve]
DNS=180.76.76.76 8.8.8.8
```

# SOLUTIONS

## DISABLE SWAP

```ini
; .wslconfig
[wsl12]
swap=0
```