---
title: vim插件——gundo
tags: vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《gundo 官方文档》](https://github.com/sjl/gundo "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



# 1 简介

* **插件介绍**：查看版本历史，并还原选定版本。
* **仓库地址**：<https://github.com/sjl/gundo>

# 2 安装
* `$vim ~/.vimrc`
* 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'sjl/gundo'`
* `:wq`
* `$vim`
* `:PluginInsttall`

# 3 使用                                        
* 在`~/.vimrc`添加`nnoremap <快捷键名> :GundoToggle<CR>`，以使用快捷键来显示和关闭插件窗口
* 默认情况下按`j或K`键在不同版本之间移动，`gg`跳到开头，`G`跳到结尾
* 按`p`键可以打开一个预览窗口以显示当前版本和选中版本之间的差异
* 按`q`键关闭gundo窗口


* 在版本树中，当前版本用`@`表示，历史版本用`O`表示。
* 当打开插件窗口（不是打开插件，插件在后台运行），光标停在当前版本。
* 当在不同版本之间移动时，gundo会显示不同版本的差异
* 按`ENTER`键还原到选定的版本


# 4 选项
* `let g:gundo_width = 45`
 设置gundo窗口和预览窗口的宽度，默认为45
 
* `let g:gundo_preview_height = 15`
 预览窗口的高度，默认为15
 
* `let g:gundo_preview_bottom = 0`
 强制预览窗口位于gundo窗口下方，默认禁用该选项
 
* `let g:gundo_right = 0`
 gundo窗口在右边，  默认禁用该选项

* `let g:gundo_help = 1 `
 在gundo窗口显示帮助信息，默认启用该选项
 
* `let g:gundo_disable = 0`
 禁用gundo，默认为禁用该选项            
 
* `let g:gundo_map_move_older = "j"`
 老一版按键，默认为j
 
* `let g:gundo_map_move_newe ="k"`
 新一版按键，默认为k
 
* `let g:gundo_close_on_revert = 0`
 还原版本后，自动关闭窗口，默认禁用该选项    
 
* `let g:gundo_preview_statusline = ""`
 设置预览窗口状态栏，默认为空
 
* `let g:gundo_tree_statusline = ""`                            
 设置gundo窗口状态栏，默认为空
 
* `let g:gundo_auto_preview  = 1`                               
 在不同版本间移动时自动显示差异，默认启用该选项
 
* `let g:gundo_playback_delay = 60`    
 设置回放延迟，默认60毫秒
 
* `let g:gundo_return_on_revert = 1 `  
  还原版本后保持Gundo窗口中的焦点，默认启用该选项
  
* `let g:gundo_prefer_python3 = 0`                             
   使用pytho3作为解释器，默认禁用该选项


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《gundo 官方文档》](https://github.com/sjl/gundo "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
