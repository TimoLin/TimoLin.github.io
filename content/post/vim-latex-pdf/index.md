---
title: GVIM下的LaTeX编译实时预览
description: Gvim+LaTeX+Okular
date: 2024-02-01 00:00:00+0000
image: cover.png
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
categories:
  - Linux
tags:
  - Ubuntu
  - vim
  - LaTeX
---

## 说明
- 编辑器：GVIM-9.0
- LaTeX插件：[vim-tex](https://github.com/lervag/vimtex)或者[vim-latex](https://github.com/vim-latex/vim-latex)
- PDF阅读器：[Okular](https://okular.kde.org/)

本方案参考了[StackExchange Answer](https://tex.stackexchange.com/a/531555)

## 1. LaTeX编译时打开SyncTeX支持
编译时打开SyncTeX支持才能够在tex与PDF之间进行跳转。 
例如使用`pdflatex`时：
```sh
pdflatex -synctex=1 file.tex
```
或者使用`latexmk`时：
```
latexmk -xelatex -synctex=1 file.tex
```
上述选项会生成一个`file.synctex.gz`文件。
## 2. Tex源码定位到PDF（Tex-to-PDF）
Okular支持打开PDF时指定到行位置，将下面的函数添加到`~/.vimrc`中：

对于`vim-tex`插件：
```vimrc
function! OkularFind()
    let this_tex_file = expand('%:p')
    let master_tex_file = b:vimtex.tex
    let pdf_file = fnamemodify(master_tex_file, ':p:r') . '.pdf'
    let line_number = line('.')
    let okular_cmd = 'okular --noraise --unique "' . pdf_file . '#src:' . line_number . ' ' . this_tex_file . '"'
    let s:okular_job = job_start(['/bin/bash', '-c', okular_cmd])
endfunction
nnoremap <leader>f :call OkularFind()<cr>
```
对于`vim-latex`插件：
```vimrc
function! OkularFind()
    let this_tex_file = expand('%:p')
    let master_tex_file = b:vimtex.tex
    let pdf_file = fnamemodify(master_tex_file, ':p:r') . '.pdf'
    let line_number = line('.')
    let okular_cmd = 'okular --noraise --unique "' . pdf_file . '#src:' . line_number . ' ' . this_tex_file . '"'
    let s:okular_job = job_start(['/bin/bash', '-c', okular_cmd])
endfunction
nnoremap <leader>f :call OkularFind()<cr>
```
其中，二者的区别是，对于多文件的项目如何找到Tex项目的主文件：
```vimrc
# vim-tex
let master_tex_file = b:vimtex.tex
# vim-latex
let master_tex_file = Tex_GetMainFileName()

```
在`gvim`下将光标移动到想要预览的内容，通过`\f`来打开`Okular`并指定到要预览的内容。


## 3. PDF定位到Tex源码（PDF-To-Tex）
在`Okular`中设置以下：
1. `设置` - `Settings`
2. `配置Okular` - `Configure Okular`
3. `编辑器` - `Editor`
4. `选择自定义文本编辑器` - `Custom Text Editor`
5. `在命令框中填入下列命令` - `Input the following command`
   ```sh
   gvim --remote +%l %f
   ```
在浏览模式下，使用快捷键`Shift+鼠标左键`单击想要查看源码的内容，即可跳转到相应的Tex源码文件位置中。

