BPF(Berkeley Packet Filter) 使用
================================

BPF可以将filter放到kernel space，这样可以减少user
space的cpu负载。使用该功能需要包含头文件\ ``#include <linux/filter.h>``

BSP指令使用类似汇编的伪代码，并运行在一个简单的虚拟机上。

--------------

创建filter 指令
---------------

BSP的指令写起来比较复杂，好在我们可以使用\ ``tcpdump -dd``\ 的命令来生成这样的指令。
关于tcpdump中的过滤命令可以参考\ `1 <http://www.tcpdump.org/manpages/pcap-filter.7.html>`__\ 。

| 例如，下面的命令是用来在抓取802.11帧的时候的过滤条件：
| 1. ``nods``\ 或者\ ``tods``\ 的包；
| 2. 并且包的类型为部分data包和部分mgt包，主要都是由设备发往AP的。

::

   tcpdump -i wlan0-monitor -e -d \(dir nods or dir tods\) and \(\(type data subtype data or subtype data-cf-ack or subtype data-cf-poll or subtype data-cf-ack-poll or subtype null or subtype cf-ack or subtype cf-poll or subtype cf-ack-poll\) or \(type mgt subtype assoc-req or subtype reassoc-req or subtype probe-req or subtype atim or subtype disassoc or subtype auth or subtype deauth\)\)

| 使用上述的\ ``tcpdump -d``\ 命令后得到的可读的BPF指令为：
| 具体每个指令的含义可以参考\ `2 <https://www.kernel.org/doc/Documentation/networking/filter.txt>`__\ 。

::

   (000) ldb      [3]
   (001) lsh      #8
   (002) tax
   (003) ldb      [2]
   (004) or       x
   (005) tax
   (006) ldb      [x + 1]
   (007) jset     #0x3             jt 8    jf 10
   (008) and      #0x3
   (009) jeq      #0x1             jt 10   jf 51
   (010) ldb      [x + 0]
   (011) and      #0xfc
   (012) jeq      #0x8             jt 52   jf 13
   (013) ldb      [x + 0]
   (014) and      #0xfc
   (015) jeq      #0x18            jt 52   jf 16
   (016) ldb      [x + 0]
   (017) and      #0xfc
   (018) jeq      #0x28            jt 52   jf 19
   (019) ldb      [x + 0]
   (020) and      #0xfc
   (021) jeq      #0x38            jt 52   jf 22
   (022) ldb      [x + 0]
   (023) and      #0xfc
   (024) jeq      #0x48            jt 52   jf 25
   (025) ldb      [x + 0]
   (026) and      #0xfc
   (027) jeq      #0x58            jt 52   jf 28
   (028) ldb      [x + 0]
   (029) and      #0xfc
   (030) jeq      #0x68            jt 52   jf 31
   (031) ldb      [x + 0]
   (032) and      #0xfc
   (033) jeq      #0x78            jt 52   jf 34
   (034) ldb      [x + 0]
   (035) jset     #0xfc            jt 36   jf 52
   (036) and      #0xfc
   (037) jeq      #0x20            jt 52   jf 38
   (038) ldb      [x + 0]
   (039) and      #0xfc
   (040) jeq      #0x40            jt 52   jf 41
   (041) ldb      [x + 0]
   (042) and      #0xfc
   (043) jeq      #0x90            jt 52   jf 44
   (044) ldb      [x + 0]
   (045) and      #0xfc
   (046) jeq      #0xa0            jt 52   jf 47
   (047) ldb      [x + 0]
   (048) and      #0xfc
   (049) jeq      #0xb0            jt 52   jf 50
   (050) jeq      #0xc0            jt 52   jf 51
   (051) ret      #0
   (052) ret      #65535

使用\ ``tcpdump -dd``\ 命令可以得到C语言中使用的指令，如下：

::

   { 0x30, 0, 0, 0x00000003 },
   { 0x64, 0, 0, 0x00000008 },
   { 0x7, 0, 0, 0x00000000 },
   { 0x30, 0, 0, 0x00000002 },
   { 0x4c, 0, 0, 0x00000000 },
   { 0x7, 0, 0, 0x00000000 },
   { 0x50, 0, 0, 0x00000001 },
   { 0x45, 0, 2, 0x00000003 },
   { 0x54, 0, 0, 0x00000003 },
   { 0x15, 0, 41, 0x00000001 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 39, 0, 0x00000008 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 36, 0, 0x00000018 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 33, 0, 0x00000028 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 30, 0, 0x00000038 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 27, 0, 0x00000048 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 24, 0, 0x00000058 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 21, 0, 0x00000068 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 18, 0, 0x00000078 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x45, 0, 16, 0x000000fc },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 14, 0, 0x00000020 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 11, 0, 0x00000040 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 8, 0, 0x00000090 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 5, 0, 0x000000a0 },
   { 0x50, 0, 0, 0x00000000 },
   { 0x54, 0, 0, 0x000000fc },
   { 0x15, 2, 0, 0x000000b0 },
   { 0x15, 1, 0, 0x000000c0 },
   { 0x6, 0, 0, 0x00000000 },
   { 0x6, 0, 0, 0x0000ffff }

将filter attach到socket上
-------------------------

::

   static struct sock_filter BPF_code[] = {
       { 0x30, 0, 0, 0x00000003 },
       { 0x64, 0, 0, 0x00000008 },
       { 0x7, 0, 0, 0x00000000 },
       { 0x30, 0, 0, 0x00000002 },
       { 0x4c, 0, 0, 0x00000000 },
       { 0x7, 0, 0, 0x00000000 },
       { 0x50, 0, 0, 0x00000001 },
       { 0x45, 0, 2, 0x00000003 },
       { 0x54, 0, 0, 0x00000003 },
       { 0x15, 0, 41, 0x00000001 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 39, 0, 0x00000008 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 36, 0, 0x00000018 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 33, 0, 0x00000028 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 30, 0, 0x00000038 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 27, 0, 0x00000048 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 24, 0, 0x00000058 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 21, 0, 0x00000068 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 18, 0, 0x00000078 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x45, 0, 16, 0x000000fc },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 14, 0, 0x00000020 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 11, 0, 0x00000040 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 8, 0, 0x00000090 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 5, 0, 0x000000a0 },
       { 0x50, 0, 0, 0x00000000 },
       { 0x54, 0, 0, 0x000000fc },
       { 0x15, 2, 0, 0x000000b0 },
       { 0x15, 1, 0, 0x000000c0 },
       { 0x6, 0, 0, 0x00000000 },
       { 0x6, 0, 0, 0x0000ffff }
   };

       struct sock_fprog kernel_filter;

       /* create kernel space filter */
       kernel_filter.len = sizeof(BPF_code) / sizeof(BPF_code[0]);
       kernel_filter.filter = BPF_code;

       ret = setsockopt(s_capture_sock, SOL_SOCKET, SO_ATTACH_FILTER, &kernel_filter,
               sizeof(kernel_filter));
       if (ret == -1) {
           syslog(LOG_ERR, "(%s): attacth filter to capture socket failed, %d", __func__, errno);
           reset_kernel_filter();
       }

::

   static int
   reset_kernel_filter()
   {
       /*
        * setsockopt() barfs unless it get a dummy parameter.
        * valgrind whines unless the value is initialized,
        * as it has no idea that setsockopt() ignores its
        * parameter.
        */
       int dummy = 0;

       return setsockopt(s_capture_sock, SOL_SOCKET, SO_DETACH_FILTER,
                      &dummy, sizeof(dummy));
   }

这样就可以在kernel space中开启过滤功能了。

**使用了BPF之后，发现user space的用户进程cpu负载明显降低**

参考文献：
----------

| 【1】http://www.tcpdump.org/manpages/pcap-filter.7.html
| 【2】https://www.kernel.org/doc/Documentation/networking/filter.txt
