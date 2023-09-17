---
title: 我的Linux
description: 介绍下我的Linux开发环境搭建
date: 2023-09-16
image: linux.jpg
categories:
  - Linux
tags:
  - Linux
  - zsh
  - vim
  - OpenFOAM
weight: 1
---

## 更换国内源 & 更新
[中科大软件源](https://mirrors.ustc.edu.cn/help/ubuntu.html)

```sh
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sudo apt update
sudo apt dist-upgrade
```
## 安装软件
```sh
# 更换为zsh终端
sudo apt install zsh autojump 
sudo apt install vim ctags
sudo apt install build-essential
# 代理
sudo apt install proxychains
```

## 配置
### 配置Zsh: [oh-my-zsh](https://ohmyz.sh/#install)
将默认bash更换为zsh:
```sh
chsh -s /bin/zsh
```
安装`oh-my-zsh`：
```sh
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```
### 配置代理: proxychains
`Proxychains`可以使任何程序通过代理上网，给终端环境下的命令带来了很多便利。
修改`/etc/proxychains.conf`末尾行：
```sh
socks4 127.0.0.1 9095
```
为（此处假设10808为Clash或V2Ray的代理端口）：
```sh
socks5 127.0.0.1 10808
```
终端命令使用代理：
```sh
# 普通命令
proxychains wget **
# sudo 命令
sudo proxychains apt update
sudo proxychains apt distupgrade
```

### VIM配置

更换最新的Vim发行仓库并更新Vim
```sh
sudo add-apt-repository ppa:jonathonf/vim
sudo apt update
sudo apt install vim
```

#### VIM-Plug插件管理器
```sh
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```
#### [coc.nvim](https://github.com/neoclide/coc.nvim)
(1) 安装coc依赖
```sh
# C/C++ lsp 
sudo snap install ccls
sudo apt install bear 
# Fortran lsp
pip3 install fortran-language-server 
```
插件安装，在vim命令模式下输入：
```
:CocInstall coc-pyright
```
(2) C/C++程序生成 `compile_commands.json`
1.  对采用`cmake`编译的程序，例如 `FlameMaster`
    ```sh
    # Recompile with `-DCMAKE_EXPORT_COMPILE_COMMANDS=1 option`
    cmake ../Repository -DCMAKE_BUILD_TYPE=Release -DCMAKE_EXPORT_COMPILE_COMMANDS=1
    cmake --build . --parallel --target install --config Release
    cp compile_commands.json ../Repository/
    ```
2. 对直接采用 `make` 编译的程序，例如 `OpenFOAM`

   参考 [教程](https://openfoamwiki.net/index.php/HowTo_Use_OpenFOAM_with_Visual_Studio_Code).

   对较低版本的OpenFOAM，修改 OpenFOAM-2.3.1/wmake/wmake:
   ```diff
    +#------------------------------------------------------------------------------
    +# check if bear is installed and we are not already running under bear
    +#------------------------------------------------------------------------------
    +
    +if [ -z "${RUNNING_UNDER_BEAR}" ] ; then
    +    if ! bear --version > /dev/null ; then
    +        echo "WARNING: bear is not installed -> There will be no compile_commands.json output." 1>&2
    +    elif printf '%s\n%s\n' "bear 3.0.0" "$(bear --version)" | sort -V -C ; then
    +        #bear version >= 3.0.0
    +        export RUNNING_UNDER_BEAR=true
    +        mkdir -p $FOAM_LIBBIN
    +        bear --append --output $FOAM_LIBBIN/../compile_commands.json -- wmake $@
    +        exit $?
    +    elif printf '%s\n%s\n' "bear 2.0.0" "$(bear --version)" | sort -V -C ; then
    +        #bear version >= 2.0.0
    +        export RUNNING_UNDER_BEAR=true
    +        mkdir -p $FOAM_LIBBIN
    +        bear --append -o $FOAM_LIBBIN/../compile_commands.json wmake $@
    +        exit $?
    +    else
    +        echo "WARNING: bear version is below 2.0.0 -> There will be no compile_commands.json output." 1>&2
    +    fi
    +fi
    
    ```

   在 `OpenFOAM-2.3.1/bin/tools` 中添加[`vscode-settings`](https://develop.openfoam.com/Development/openfoam/-/blob/a50047bbcc9ee270ebddd6e95ea7d0e01f2a525f/bin/tools/vscode-settings)。

   Generate VSCode setting files:
   ```sh
   cd $WM_PROJECT_DIR
   mkdir .vscode
   ./bin/tools/vscode-setting > .vscode/settings.json
   # Clean OpenFOAM installation
   wclean all
   # Re-build
   ./Allwmake
   ```
   创建软链接： `VIM`/`coc.nvim`:
   ```sh
   cd $WM_PROJECT_DIR
   ln -s ~/OpenFOAM/OpenFOAM-2.3.1/platforms/linux64GccDPOpt/compile_commands.json ./compile_commands.json
   ```
(4) 附录:
1. `coc-settings.json`(位于 `~/.vim`下):
    ```json
    {
        "languageserver": {
            "ccls": {
                "command": "ccls",
                "filetypes": ["c", "cc", "cpp", "c++", "objc", "objcpp"],
                "rootPatterns": [".ccls", "compile_commands.json", ".git/", ".root"],
                "initializationOptions": {
                    "cache": {
                        "directory": ".cache/ccls"
                    },
                    "highlight": {"lsRanges": true }
                }
            },
            "fortran": {
                "command": "fortls",
                "filetypes": ["fortran"],
                "rootPatterns": [".fortls", ".git/"]
            }
    }
    ```
