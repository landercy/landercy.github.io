# install

断网安装，否则安装到一半更新库时会卡住


# INIT

```sh
# https://mirrors.tuna.tsinghua.edu.cn/help/msys2/
sed -i "s#https\?://mirror.msys2.org/#https://mirrors.tuna.tsinghua.edu.cn/msys2/#g" /etc/pacman.d/mirrorlist*

pacman -Syu
pacman -S git man vim zsh

# https://ohmyz.sh/#install
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## TRUST ROOT CA

```sh
/etc/pki/ca-trust/source/anchors
update-ca-trust
```

## INTEGRATION

### VSCODE

```json
"terminal.integrated.profiles.windows": {
    "zsh": {
    "path": "C:/Users/cn407ff/Raymond/apps/msys2/usr/bin/zsh.exe",
    "args": [
        "-l",
        "-i",
    ]
    }
},
"terminal.integrated.defaultProfile.windows": "zsh"
```

### TERMINAL / IDEA

```
C:/Users/cn407ff/Raymond/apps/msys2/usr/bin/zsh.exe -l -i
```

## APPS

### PYTHON

```sh
pacman S python python-pip
```

### NODEJS

https://packages.msys2.org/search?t=pkg&q=node
Binary Packages: mingw-w64-ucrt-x86_64-nodejs

```sh
# nodejs
export PATH="$PATH:/c/Users/cn407ff/Raymond/apps/node"
```

### SQLITE

```sh
pacman -S sqlite3 mingw-w64-x86_64-sqlcipher
```