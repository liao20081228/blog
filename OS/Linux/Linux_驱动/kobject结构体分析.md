---
title: kobject结构体分析
tags: Linux驱动
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 1 简介
&emsp;&emsp;随着Linux内核的发展壮大，其支持的设备也越来越多，但一直没有一个很好的方法来管理慢慢增多的设备驱动。为了能够在内核中提供统一的机制来对设备进行分类，同时在更高的功能层面上描述这些设备，并使得这些设备对用户空间可见。从2.6开始，Linux内核引入一个新的设备模型来对系统结构做一般性的抽象描述，可以用于支持不同的任务需求，并提供了用户空间访问的接口。

&emsp;&emsp;Kobject是linux设备驱动模型的基础，也是设备模型中抽象的一部分。如果想了解设备驱动模型就需要明白Kobject的构成或原理。linux内核为了兼容各种形形色色的设备，就需要对各种设备的共性进行抽象，抽象出一个基类，其余的设备只需要继承此基类就可以了。而此基类就是kobject，但是C语言没有面向对象语法，这时候就需要将此基类(kobject)嵌入到具体的结构体中（作为其结构体成员），从而就可以访问控制此设备的操作。通常驱动程序员很少使用到kobject结构及其相关接口，而是使用封装之后的更高层的接口函数。


&emsp;&emsp;kobject是组成设备device、驱动driver、总线bus、class的基本结构。如果把前者看成基类，则后者均为它的派生产物。device、driver、bus、class构成了设备模型，而kobject内嵌于其中，将这些设备模型的部件组织起来，并形成了sysfs文件系统。kobject就是device、driver、bus、class在sysfs文件系统中的代表。在sysfs操作设备时，也必须通过kobject这个中间人来完成。kobject的主要功能如下：
* **对象的引用计数**：通常一个内核对象被创建时，不可能知道该对象存活的时间。跟踪此对象生命周期的一个方法是使用引用计数。当内核中没有代码持有该对象的引用时，该对象将结束自己的有效生命周期，并且可以被删除。
* **sysfs表述**：在sysfs中显示的每一个对象，都对应一个kobject，它被用来与内核交互并创建它的可见表述。
* **数据结构关联**：从整体上看，设备模型是一个友好而复杂的数据结构，通过在其间的大量连接而构成一个多层次的体系结构。Kobject实现了该结构并把它们聚合在一起。
* **uevent事件处理**：当系统中的硬件被热插拔时，在kobject子系统控制下，将产生事件以通知用户空间。

# 2 kobject分析
```c
/*位于include/linux/kobject.h*/
struct kobject {
	const char		*name;
	struct list_head	entry;
	struct kobject	*parent;
	struct kset		*kset;
	struct kobj_type	*ktype;
	struct kernfs_node *sd;
	struct kref		kref;
#ifdef CONFIG_DEBUG_KOBJECT_RELEASE
	struct delayed_work	release;
#endif
	unsigned int state_initialized:1;
	unsigned int state_in_sysfs:1;
	unsigned int state_add_uevent_sent:1;
	unsigned int state_remove_uevent_sent:1;
	unsigned int uevent_suppress:1;
}；

/*
name: 用来表示内核对象的名称，如果该内核对象加入到系统，则对应着sysfs下的一个同名文件夹。

entry: 双向链表指针，用于将同一kset集合中的kobject链接到一起，便于访问

parent: 用来指向该内核对象的上层节点，从而可以实现内核对象的层次化结构，在sysfs表现为上一级目录

kset: 用来执行内核对象所属的kset。kset对象用来容纳一系列同类型的kobject

ktype: 用来定义该内核对象的sys文件系统的相关操作函数和属性。

sd: 用来表示该内核对象在sys文件系统中的目录项实例

kref: 其核心是原子操作变量，用来表示该内核对象的引用计数。

state_initialized: 用来表示该内核对象的初始化状态，1表示已经初始化，0表示未初始化。

state_in_sysfs: 用来表示该内核对象是否在sys中已经存在，已存在则置1，否则为0。。

state_add_uevent_sent: 用来表示该内核对象是否向用户空间发送了ADD uevent事件

state_remove_uevent_sent:用来表示该内核对象是否向用户空间发送了Remove uevent事件

uevent_suppress: 用来表示该内核对象状态发生改变时，时候向用户空间发送uevent事件，1表示不发送。
*/

/* 双向链表位于 drivers/gpu/drm/nouveau/include/nvif/list.h */
struct list_head{
	struct list_head *next, *prev;
};

/*位于include/linux/kobject.h*/
struct kset {  
	struct list_head list;
	spinlock_t list_lock;
	struct kobject kobj;
	const struct kset_uevent_ops *uevent_ops;
} __randomize_layout;

/*位于include/linux/kobject.h*/
struct kobj_type {
	void (*release)(struct kobject *kobj);
	const struct sysfs_ops *sysfs_ops;
	struct attribute **default_attrs;
	const struct kobj_ns_type_operations *(*child_ns_type)(struct kobject *kobj);
	const void *(*namespace)(struct kobject *kobj);
};

/*位于include/linux/kref.h*/
struct kref {
	refcount_t refcount;
};
```
kobject数据结构通常的用法就是嵌入到某一个对象的数据结构中，比如struct cdev结构:
```c
struct cdev {
	struct kobject kobj;
	struct module *owner;
	const struct file_operations *ops;
	struct list_head list;
	dev_t dev;
	unsigned int count;
} __randomize_layout;
```

&emsp;&emsp;kobject结构在sysfs中的入口是一个目录，因此添加一个kobject的动作也会导致在sysfs中新建一个对应kobject的目录，目录名是 `kobject.name`。该入口目录位于kobject的parent指针的目录当中，若parent指针为空，则将parent设置为该kobject中的kset对应的kobject(&kobj->kset->kobj)。如果parent指针和kset都是NULL，此时sysfs的入口目录将会在顶层目录下(/sys)被创建，不过不建议这么做。详细的创建目录过程可以看后面介绍的kobject_init_and_add函数的介绍。



# 3 相关函数
```c?linenums=false
//初始化kobject 以便可以通过kobject_add（）调用;
void kobject_init(struct kobject *kobj, struct kobj_type *ktype)；
 
 
//将kobj 对象加入sysfs
int kobject_add(struct kobject *kobj, struct kobject *parent,const char *fmt, ...) ;


//kobject_init() and kobject_add()函数的结合，返回值与kobject_add（）相同；与kobject_create_and_add的区别是，kobject结构体必须已经创建好，动态创建或者静态声明均可
int kobject_init_and_add(struct kobject *kobj, struct kobj_type *ktype,struct kobject *parent, const char *fmt, ...);


//从Linux 设备层次(hierarchy)中删除kobj 对象;
void kobject_del(struct kobject *kobj)


//动态的创建一个kobject结构体；
struct kobject *kobject_create(void)


//动态创建了一个kobject结构体，将其初始化，将其加入到kobject层次中，并最终返回所创建的 kobject的指针，当然如果函数执行失败，则返回NULL；
struct kobject *kobject_create_and_add(const char *name, struct kobject *parent);


//改变一个kobject的名字;
int kobject_rename(struct kobject *kobj, const char *new_name);
	

//将一个kobject从一个层次移动到另一个层次;
int kobject_move(struct kobject *kobj, struct kobject *new_parent);


//将kobj 对象的引用计数加1，同时返回该对象的指针;
struct kobject *kobject_get(struct kobject *kobj);


//将kobj 对象的引用计数减1，如果引用计数降为0，则调用kobject_release()释放该kobject 对象;
void kobject_put(struct kobject *kobj);


//返回kobject的路径；
char *kobject_get_path(struct kobject *kobj, gfp_t gfp_mask);


//设置kobject的名字
int kobject_set_name(struct kobject *kobj, const char *fmt, ...);
```




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
