# python3-v4l2capture

python3通过v4l2获取图像数据适用的库

## 关于Jetson

NVIDIA Jetson因为其硬件以及软件都是经过NV定制化的，因此可能与通用的Linux不太相同，
对于v4l2这个Linux下的视频设备框架尤其是如此。

目前发现的问题有：Jetson下尽量不要使用`libv4l-dev`包提供的`v4l2_ioctl`函数，
而应该使用系统的`ioctl`函数，`v4l2capture`这个包源码本身默认使用的是前者，
因此目前对他的源码进行了修改。


## 关于V4L2

V4L2是Video For Linux 2的简称，它是Linux下用于管理视频流设备的设备驱动框架，
V4L2很强大，涵盖非常多内容，但也同时导致它很繁杂，这里只做简单介绍。

[V4L2官方手册](https://www.kernel.org/doc/html/v4.9/media/uapi/v4l/v4l2.html)

- `v4l2-ctl` 用于进行V4L2设备的设置与控制的命令行工具，常用的命令如下：
	- `v4l2-ctl --list-devices` 列出所有的v4l2设备概要
	- `v4l2-ctl --device /dev/video0 --list-formats-ext`列出`/dev/video0`设备的支持的扩展格式
	- `v4l2-ctl --device /dev/video0 --list-ctrls`列出`/dev/video0`设备的支持的控制参数（注意这里列出来的不一定就能够正常工作，只是说明驱动支持而已）
- `v4l2-compliance --device /dev/video0`测试设备的控制兼容性，具体每个控制有什么作用需要具体查阅V4L2的手册

## 示例

```python3
import os
import select
import v4l2capture

if __name__ == '__main__':

    width = 1920
    height = 1080

    video = v4l2capture.Video_device("/dev/video0")
	# 该示例中，相机参数设置为，1920x1080分辨率的，格式为RG10
	# 注意：
    video.set_format(width,height, fourcc="RG10")
	# 该示例中，相机不支持设置FPS, 支持设置需要设备支持VIDIOC_S_PARM
    # video.set_fps(10)
    video.create_buffers(10)
    video.queue_all_buffers()

    video.start()

	# 这里使用的是poll，使用linux的selete方法也是可以的
    pollerObject = select.poll()
    pollerObject.register(video, select.POLLIN)
    counter = 1;
    while True:
        print("reading a image %u" % counter )
        fdVsEvent = pollerObject.poll(1000)
        for descriptor, Event in fdVsEvent:
			# image_data是原始的字节数据
            image_data = video.read_and_queue()
            print("read a image %u" % counter )
            counter = counter + 1
 
    video.close()
```