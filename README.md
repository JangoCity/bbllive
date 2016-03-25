# bbllive
支持rtmp协议流媒体服务器，初步测试30000+，供学习用go写高并发高性能程序的同学参考
go1.6+
### 思想
- 通过消息通知共享数据，锁的开销没想象的那么大
- 对象复用，减少垃圾收集
- 减少系统调用，通过数据包合并一次尽量发送多帧，本程序以GOP对齐后一次发送
- 减少对象复制，本程序没有彻底实现，应该还有优化空间

###还可以优化的地方
- 读消息时一次系统调用可以读取更多消息
- 减少数据包复制，可以通过引用计数实现
- ffmpeg推流时因为通过两个chunkid发送过来，服务器端计算时间戳时有误差，所以建议还是用专业的编码器推流

### 测试环境：
   2颗E5-2650 v2，64G内存，单机测试（因为没有万兆网络环境）
   推流工具xsplit2.7，关键帧间隔2秒，混合码流900kbps，CBR
   负载工具 srs-bench/sb_rtmp_load_fast
   每次5000并发，共跑6个sb_rtmp_load_fast进程，每进程占用100%
   因为是本机测试，所以软中断和负载程序占用了很大一部分CPU，所以有万兆环境+多队列万兆网卡的性能应该还可以进一步提升
   测试时通过vlc观看不卡，中间关闭后再开也不卡，多个vlc播放器之间延时几乎没有

### 30000的时候系统负载
```
top - 13:18:15 up 16 days, 23:42, 11 users,  load average: 76.44, 56.27, 53.53
Tasks: 942 total,   3 running, 935 sleeping,   4 stopped,   0 zombie
Cpu0  : 19.6%us, 15.4%sy,  0.0%ni, 61.8%id,  0.0%wa,  0.0%hi,  3.3%si,  0.0%st
Cpu1  : 24.9%us, 13.4%sy,  0.0%ni, 57.4%id,  0.0%wa,  0.0%hi,  4.3%si,  0.0%st
Cpu2  : 17.8%us, 14.5%sy,  0.0%ni, 64.7%id,  0.0%wa,  0.0%hi,  3.0%si,  0.0%st
Cpu3  : 20.6%us, 12.0%sy,  0.0%ni, 64.2%id,  0.0%wa,  0.0%hi,  3.2%si,  0.0%st
Cpu4  : 17.4%us, 13.1%sy,  0.0%ni, 66.7%id,  0.0%wa,  0.0%hi,  2.8%si,  0.0%st
Cpu5  : 15.3%us, 23.1%sy,  0.0%ni, 59.0%id,  0.0%wa,  0.0%hi,  2.6%si,  0.0%st
Cpu6  : 16.5%us, 13.3%sy,  0.0%ni, 67.0%id,  0.0%wa,  0.0%hi,  3.2%si,  0.0%st
Cpu7  : 18.9%us, 32.4%sy,  0.0%ni, 45.6%id,  0.0%wa,  0.0%hi,  3.0%si,  0.0%st
Cpu8  : 20.3%us, 14.1%sy,  0.0%ni, 61.6%id,  0.0%wa,  0.0%hi,  4.0%si,  0.0%st
Cpu9  : 23.4%us, 11.7%sy,  0.0%ni, 62.3%id,  0.0%wa,  0.0%hi,  2.6%si,  0.0%st
Cpu10 : 18.9%us, 15.6%sy,  0.0%ni, 62.2%id,  0.0%wa,  0.0%hi,  3.3%si,  0.0%st
Cpu11 : 19.4%us, 14.1%sy,  0.0%ni, 63.4%id,  0.0%wa,  0.0%hi,  3.2%si,  0.0%st
Cpu12 : 31.3%us, 11.9%sy,  0.0%ni, 54.0%id,  0.0%wa,  0.0%hi,  2.9%si,  0.0%st
Cpu13 : 22.0%us, 10.8%sy,  0.0%ni, 63.9%id,  0.0%wa,  0.0%hi,  3.3%si,  0.0%st
Cpu14 : 17.1%us, 16.2%sy,  0.0%ni, 63.8%id,  0.0%wa,  0.0%hi,  2.9%si,  0.0%st
Cpu15 : 20.3%us, 19.6%sy,  0.0%ni, 57.0%id,  0.0%wa,  0.0%hi,  3.1%si,  0.0%st
Cpu16 : 25.9%us, 20.3%sy,  0.0%ni, 50.7%id,  0.0%wa,  0.0%hi,  3.1%si,  0.0%st
Cpu17 : 17.2%us, 28.8%sy,  0.0%ni, 52.1%id,  0.0%wa,  0.0%hi,  1.9%si,  0.0%st
Cpu18 : 23.8%us, 35.2%sy,  0.0%ni, 38.1%id,  0.0%wa,  0.0%hi,  2.9%si,  0.0%st
Cpu19 : 26.6%us, 13.8%sy,  0.0%ni, 56.2%id,  0.0%wa,  0.0%hi,  3.4%si,  0.0%st
Cpu20 : 21.5%us, 13.8%sy,  0.0%ni, 61.9%id,  0.0%wa,  0.0%hi,  2.8%si,  0.0%st
Cpu21 : 18.4%us, 25.6%sy,  0.0%ni, 53.1%id,  0.0%wa,  0.0%hi,  3.0%si,  0.0%st
Cpu22 : 14.9%us, 33.7%sy,  0.0%ni, 49.3%id,  0.0%wa,  0.0%hi,  2.2%si,  0.0%st
Cpu23 : 18.4%us, 12.3%sy,  0.0%ni, 65.8%id,  0.0%wa,  0.0%hi,  3.5%si,  0.0%st
Cpu24 : 19.3%us, 13.1%sy,  0.0%ni, 64.9%id,  0.0%wa,  0.0%hi,  2.6%si,  0.0%st
Cpu25 : 20.6%us, 22.2%sy,  0.0%ni, 54.7%id,  0.0%wa,  0.0%hi,  2.6%si,  0.0%st
Cpu26 : 17.9%us, 50.0%sy,  0.0%ni, 29.8%id,  0.0%wa,  0.0%hi,  2.3%si,  0.0%st
Cpu27 : 15.4%us, 27.1%sy,  0.0%ni, 55.6%id,  0.0%wa,  0.0%hi,  1.9%si,  0.0%st
Cpu28 : 31.7%us,  9.9%sy,  0.0%ni, 55.8%id,  0.0%wa,  0.0%hi,  2.6%si,  0.0%st
Cpu29 : 34.8%us, 11.8%sy,  0.0%ni, 50.9%id,  0.0%wa,  0.0%hi,  2.4%si,  0.0%st
Cpu30 : 20.2%us, 14.7%sy,  0.0%ni, 62.1%id,  0.0%wa,  0.0%hi,  2.9%si,  0.0%st
Cpu31 : 21.9%us, 13.0%sy,  0.0%ni, 61.9%id,  0.0%wa,  0.0%hi,  3.3%si,  0.0%st
Mem:  65923556k total, 39068692k used, 26854864k free,   251764k buffers
Swap: 25164792k total,    65988k used, 25098804k free,  4924036k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                                                    
11589 root      20   0 22.8g  16g 4200 S 971.5 26.0 145:18.09 bbllive 
```

### 网络吞吐，峰值超过40Gbps，统计上看输出不平滑，这是正常的，因为服务器端是合并后一次发送
```
sar -n DEV 1

01:18:32 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:33 PM        lo 302942.99 302942.99 5223440.74 5223440.74      0.00      0.00      0.00
01:18:33 PM       em1    571.03    670.09    260.95    559.62      0.00      0.00      0.93
01:18:33 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:33 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:33 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:33 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:33 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:33 PM     bond0    571.03    670.09    260.95    559.62      0.00      0.00      0.93
01:18:33 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:33 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:34 PM        lo  44640.74  44640.74 767318.07 767318.07      0.00      0.00      0.00
01:18:34 PM       em1    661.73    716.05    294.10    405.78      0.00      0.00      1.23
01:18:34 PM       em2      1.23      0.00      0.10      0.00      0.00      0.00      1.23
01:18:34 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:34 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:34 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:34 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:34 PM     bond0    662.96    716.05    294.20    405.78      0.00      0.00      2.47
01:18:34 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:34 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:35 PM        lo 268190.76 268190.76 4569484.61 4569484.61      0.00      0.00      0.00
01:18:35 PM       em1    536.97    631.93    242.67    527.20      0.00      0.00      0.00
01:18:35 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:35 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:35 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:35 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:35 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:35 PM     bond0    536.97    631.93    242.67    527.20      0.00      0.00      0.00
01:18:35 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:35 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:36 PM        lo  41169.00  41169.00 699223.45 699223.45      0.00      0.00      0.00
01:18:36 PM       em1    595.00    655.00    316.24    338.96      0.00      0.00      0.00
01:18:36 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:36 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:36 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:36 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:36 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:36 PM     bond0    595.00    655.00    316.24    338.96      0.00      0.00      0.00
01:18:36 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:36 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:37 PM        lo 340989.11 340989.11 5565256.38 5565256.38      0.00      0.00      0.00
01:18:37 PM       em1    619.80    778.22    246.12    677.88      0.00      0.00      0.00
01:18:37 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:37 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:37 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:37 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:37 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:37 PM     bond0    619.80    778.22    246.12    677.88      0.00      0.00      0.00
01:18:37 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:37 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:38 PM        lo 274824.00 274824.00 3943548.07 3943548.07      0.00      0.00      0.00
01:18:38 PM       em1    683.00    746.00    354.36    543.61      0.00      0.00      0.00
01:18:38 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:38 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:38 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:38 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:38 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:38 PM     bond0    683.00    746.00    354.36    543.61      0.00      0.00      0.00
01:18:38 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:38 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:39 PM        lo    166.67    166.67    481.67    481.67      0.00      0.00      0.00
01:18:39 PM       em1    588.89    631.31    308.04    368.66      0.00      0.00      0.00
01:18:39 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:39 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:39 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:39 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:39 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:39 PM     bond0    588.89    631.31    308.04    368.66      0.00      0.00      0.00
01:18:39 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:39 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:40 PM        lo 382172.00 382172.00 6239203.48 6239203.48      0.00      0.00      0.00
01:18:40 PM       em1    562.00    677.00    215.53    585.04      0.00      0.00      0.00
01:18:40 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:40 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:40 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:40 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:40 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:40 PM     bond0    562.00    677.00    215.53    585.04      0.00      0.00      0.00
01:18:40 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:40 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:41 PM        lo  40381.91  40381.91 657320.67 657320.67      0.00      0.00      0.00
01:18:41 PM       em1    690.43    732.98    343.89    388.25      0.00      0.00      0.00
01:18:41 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:41 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:41 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:41 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:41 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:41 PM     bond0    690.43    732.98    343.89    388.25      0.00      0.00      0.00
01:18:41 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:41 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:42 PM        lo 346065.42 346065.42 6021440.04 6021440.04      0.00      0.00      0.00
01:18:42 PM       em1    532.71    620.56    227.57    510.71      0.00      0.00      0.00
01:18:42 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:42 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:42 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:42 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:42 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:42 PM     bond0    532.71    620.56    227.57    510.71      0.00      0.00      0.00
01:18:42 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:42 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:43 PM        lo   2684.85   2684.85  45764.79  45764.79      0.00      0.00      0.00
01:18:43 PM       em1    628.28    627.27    300.89    313.32      0.00      0.00      0.00
01:18:43 PM       em2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:43 PM       em3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:43 PM       em4      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:43 PM      p7p1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:43 PM      p7p2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:18:43 PM     bond0    628.28    627.27    300.89    313.32      0.00      0.00      0.00
01:18:43 PM     bond1      0.00      0.00      0.00      0.00      0.00      0.00      0.00

01:18:43 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:18:44 PM        lo 332971.29 332971.29 6172549.32 6172549.32      0.00      0.00      0.00
```

### bbllive服务器自己的并发数统计，差不多2秒一个GOP

```
2016/03/25 13:19:40 INFO Gop live/c 994 154
2016/03/25 13:19:40 INFO players live/c 30001
2016/03/25 13:19:42 INFO Gop live/c 995 153
2016/03/25 13:19:42 INFO players live/c 30001
2016/03/25 13:19:44 INFO Gop live/c 996 144
2016/03/25 13:19:44 INFO players live/c 30001
2016/03/25 13:19:46 INFO Gop live/c 997 154
2016/03/25 13:19:46 INFO players live/c 30001
```
