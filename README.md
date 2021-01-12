# yza
#include <linux/init.h>
#include <linux/module.h>

#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/kfifo.h>
//定义主设备号 和从设备号
#define	VSER_MAJOR 256
#define VSER_MINOR 0
#define DEV_CNT 1
#define	VSER_DEV_NAME "vser"

MODULE_LICENSE("Dual BSD/GPL");
MODULE_AUTHOR("Mr.Zhong <1642131567@qq.com>");
MODULE_DESCRIPTION("A simple Module");
MODULE_ALIAS("Visualport");
/*init a FIFO*/
DEFINE_KFIFO(vsfifo,char,32);
/*declare a device*/
static struct cdev vsdev;
//
static int vser_open(struct inode* inode,struct file* filp)
{
	return 0;
}

static int vser_relase(struct inode* inode,struct file* filp)
{
	return 0;
}
//
static ssize_t vser_read(struct file *filp,char __user *buf,size_t count,loff_t* pos)
{
	unsigned int copied = 0;
	kfifo_to_user(&vsfifo,buf,count,&copied);//

	return copied;
}
//
static ssize_t vser_write(struct file* filp,const char __user *buf,size_t count,loff_t *pos)
{
	unsigned int copied = 0;
	/*from user space copy data to kernel space*/
	kfifo_from_user(&vsfifo,buf,count,&copied);

	return copied;
}
/*relize some file operations */
//
static struct file_operations vser_ops = {
	.owner = THIS_MODULE,
	.open = vser_open,
	.release = vser_relase,
	.read = vser_read,
	.write = vser_write,
	};

/*register device*/
static int __init vser_init(void)
{
	int ret;
	dev_t dev;
	dev = MKDEV(VSER_MAJOR,VSER_MINOR); //
	ret = register_chrdev_region(dev,DEV_CNT,VSER_DEV_NAME); //
	if(ret)
		 goto reg_err; //
	cdev_init(&vsdev,&vser_ops); //
	vsdev.owner = THIS_MODULE;//

	ret = cdev_add(&vsdev,dev,DEV_CNT);//
	if(ret)
		goto add_err;
	printk(KERN_EMERG "init KFIFO driver\n");
	return 0;

add_err:
	unregister_chrdev_region(dev,DEV_CNT);//
	
reg_err:
	return ret;
}

static void __exit vser_exit(void)
{
	dev_t dev;
	dev = MKDEV(VSER_MAJOR,VSER_MINOR);
	
	cdev_del(&vsdev);
	unregister_chrdev_region(dev,DEV_CNT);//

	printk(KERN_EMERG "exit KFIFO \n");
}
/*alias*/
module_init(vser_init); //
module_exit(vser_exit);
