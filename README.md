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























import smbus  #导入smbus模块
import math   #导入math模块


power_mgmt_1 = 0x6b 
power_mgmt_2 = 0x6c 

def read_byte(adr):  #定义读字节函数，参数是adr
    return bus.read_byte_data(address, adr) #返回读字符数据 （地址指令）


def read_word(adr): #定义读取单个字函数 WORD=2Byte
    high = bus.read_byte_data(address, adr) #
    low = bus.read_byte_data(address, adr+1)
    val = (high << 8) + low
    return val


def read_word_2c(adr): #从给定的寄存器的读取一个字并转换为二进制补码
    val = read_word(adr)
    if (val >= 0x8000):
        return -((65535 - val) + 1)
    else:
        return val


def dist(a, b): #定义一个dist函数进行数学运算
    return math.sqrt((a*a)+(b*b))  #返回运算值


def get_y_rotation(x, y, z): #计算y轴的旋转度数
    radians = math.atan2(x, dist(y, z))  #返回运算值的反正切（以弧度为单位）
    return -math.degrees(radians)  #将弧度变为角度


def get_x_rotation(x, y, z): #计算x轴的旋转度数
    radians = math.atan2(y, dist(x, z))    
    return math.degrees(radians)




#从MPU6050传感器的陀螺仪上读取数据
gyro_xout = read_word_2c(0x43)   #读取x轴方向陀螺仪传感器的测量值
gyro_yout = read_word_2c(0x45)
gyro_zout = read_word_2c(0x47)

print( "gyro_xout: ", gyro_xout, " scaled: ", (gyro_xout / 131))  # 除以131，可以得到x方向每秒的旋转度数。
print( "gyro_yout: ", gyro_yout, " scaled: ", (gyro_yout / 131))  # 除以131，可以得到y方向每秒的旋转度数。
print("gyro_zout: ", gyro_zout, " scaled: ", (gyro_zout / 131))   # 除以131，可以得到z方向每秒的旋转度数。


print("accelerometer data")
print("---------")


accel_xout = read_word_2c(0x3b) #读取x轴方向加速度传感器的测量值
accel_yout = read_word_2c(0x3d) 
accel_zout = read_word_2c(0x3f) 

accel_xout_scaled = accel_xout / 16384.0   #16384为加速度原始数据值。得到x轴每秒的旋转度数
accel_yout_scaled = accel_yout / 16384.0   #16384为加速度原始数据值。得到y轴每秒的旋转度数
accel_zout_scaled = accel_zout / 16384.0   #16384为加速度原始数据值。得到z轴每秒的旋转度数

print("accel_xout: ", accel_xout, " scaled: ", accel_xout_scaled)  #打印出x轴加速度值和每秒的旋转度数
print("accel_yout: ", accel_yout, " scaled: ", accel_yout_scaled)  #打印出x轴加速度值和每秒的旋转度数
print("accel_zout: ", accel_zout, " scaled: ", accel_zout_scaled)  #打印出x轴加速度值和每秒的旋转度数

print( "x rotation: ", get_x_rotation(accel_xout_scaled, accel_yout_scaled, accel_zout_scaled)) #打印x的旋转角度
print("y rotation: ", get_y_rotation(accel_xout_scaled, accel_yout_scaled, accel_zout_scaled))  #打印y的旋转角度
