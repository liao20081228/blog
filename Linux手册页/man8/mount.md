---
title: mount 
tags: man8,系统管理
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《mount manpages》。</font>当前版本为2.36.1，手册更新时间为2015-08。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称
mount——挂载一个文件系统。

# 总览
**mount** [**-h**|**-V**]

**mount** [**-l**] [**-t** <u>文件系统类型</u>]

**mount** **-a** [**-fFnrsvw**] [**-t** <u>文件系统类型</u>] [**-O** 可选列表]

**mount** [**-fnrsvw**] [**-o** <u>选项</u>] <u>设备</u>|<u>挂载点</u>

**mount** [**-fnrsvw**] [**-t** <u>文件系统类型</u>] [**-o** <u>选项</u>] <u>设备</u> <u>挂载点</u>

**mount** **--bind**|**--rbind**|**--move** <u>旧目录</u> <u>新目录</u>

**mount** **--make-**{**shared**|**slave**|**private**|**unbindable**|**rshared**|**rslave**|**rprivate**|**runbindable**} <u>挂载点</u>
# 说明
在 Unix 系统中可访问的所有文件都排列在一棵大树中，即文件层次结构，以 / 为根。 这些文件可以分布在多个设备上。 **mount** 命令用于将在某个设备上找到的文件系统附加到大文件树上。 相反，umount(8) 命令将再次分离它。 文件系统用于控制数据如何存储在设备上或通过网络或其他服务以虚拟方式提供。

mount 命令的标准格式是：


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《mount manpages》。</font>当前版本为8.30，手册更新时间为2015-08。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

