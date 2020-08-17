[TOC]

# linux crontab 每秒钟执行

​	在crontab给出最小时间间隔来执行命令时，最小的时间间隔是一分钟，那么如何做成每秒执行一次，可以通过sleep 命令相结合来完成。

```
* * *  *  *     /root/psu.sh
* * *  *  *     sleep 1; /root/psu.sh
* * *  *  *     sleep 2; /root/psu.sh
* * *  *  *     sleep 3; /root/psu.sh
* * *  *  *     sleep 4; /root/psu.sh
* * *  *  *     sleep 5; /root/psu.sh
* * *  *  *     sleep 6; /root/psu.sh
* * *  *  *     sleep 7; /root/psu.sh
* * *  *  *     sleep 8; /root/psu.sh
* * *  *  *     sleep 9; /root/psu.sh
* * *  *  *     sleep 10; /root/psu.sh
* * *  *  *     sleep 11; /root/psu.sh
* * *  *  *     sleep 12; /root/psu.sh
* * *  *  *     sleep 13; /root/psu.sh
* * *  *  *     sleep 14; /root/psu.sh
* * *  *  *     sleep 15; /root/psu.sh
* * *  *  *     sleep 16; /root/psu.sh
* * *  *  *     sleep 17; /root/psu.sh
* * *  *  *     sleep 18; /root/psu.sh
* * *  *  *     sleep 19; /root/psu.sh
* * *  *  *     sleep 20; /root/psu.sh
* * *  *  *     sleep 21; /root/psu.sh
* * *  *  *     sleep 22; /root/psu.sh
* * *  *  *     sleep 23; /root/psu.sh
* * *  *  *     sleep 24; /root/psu.sh
* * *  *  *     sleep 25; /root/psu.sh
* * *  *  *     sleep 26; /root/psu.sh
* * *  *  *     sleep 27; /root/psu.sh
* * *  *  *     sleep 28; /root/psu.sh
* * *  *  *     sleep 29; /root/psu.sh
* * *  *  *     sleep 30; /root/psu.sh
* * *  *  *     sleep 31; /root/psu.sh
* * *  *  *     sleep 32; /root/psu.sh
* * *  *  *     sleep 33; /root/psu.sh
* * *  *  *     sleep 34; /root/psu.sh
* * *  *  *     sleep 35; /root/psu.sh
* * *  *  *     sleep 36; /root/psu.sh
* * *  *  *     sleep 37; /root/psu.sh
* * *  *  *     sleep 38; /root/psu.sh
* * *  *  *     sleep 39; /root/psu.sh
* * *  *  *     sleep 40; /root/psu.sh
* * *  *  *     sleep 41; /root/psu.sh
* * *  *  *     sleep 42; /root/psu.sh
* * *  *  *     sleep 43; /root/psu.sh
* * *  *  *     sleep 44; /root/psu.sh
* * *  *  *     sleep 45; /root/psu.sh
* * *  *  *     sleep 46; /root/psu.sh
* * *  *  *     sleep 47; /root/psu.sh
* * *  *  *     sleep 48; /root/psu.sh
* * *  *  *     sleep 49; /root/psu.sh
* * *  *  *     sleep 50; /root/psu.sh
* * *  *  *     sleep 51; /root/psu.sh
* * *  *  *     sleep 52; /root/psu.sh
* * *  *  *     sleep 53; /root/psu.sh
* * *  *  *     sleep 54; /root/psu.sh
* * *  *  *     sleep 55; /root/psu.sh
* * *  *  *     sleep 56; /root/psu.sh
* * *  *  *     sleep 57; /root/psu.sh
* * *  *  *     sleep 58; /root/psu.sh
* * *  *  *     sleep 59; /root/psu.sh
```

​	添加完成后，可能不会马上执行，可以检查日志和重新加载信息重启服务

```
# tail -200f  /var/log/cron
# service crond reload
Reloading crond:                                           [  OK  ]
# service crond restart
Stopping crond:                                            [  OK  ]
Starting crond:                                            [  OK  ]
# tail -200f  /var/log/cron
```



## 链接地址

https://blog.csdn.net/aaroun/article/details/78892984

https://blog.csdn.net/brad_chen/article/details/50318297