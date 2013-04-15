读取CPU State
****************

iostat.c (1206 `read_uptime`) --> rd_stats.c (1868 `read_uptime`) ---> iostat.c
(1214 `read_stat_cpu`) --> rd_stats.c (58 `read_stat_cpu`)

另外学习\ **/proc/uptime**\ 中两个数字的意义：\ [#a]_

* 第一个为系统启动到现在的时间t1；
* 第二个为系统空闲时间t2（所有CPU的总空闲时间）。


如果为多核处理器，t2有可能大于t1。因为系统运行时间不会考虑CPU数目。所以如果要计\
算CPU的利用率，应该是：

.. math::

    idle\% = \frac{t_2}{t_1*ncpu}

还有文件\ **/proc/stat**\ 内容的意义：[#b]_

.. todo::
    
    添加/proc/stat文件内容的描述

关于对\ */proc/uptime*\ 时间的计算：

.. sourcecode:: c

    // rd_stats.c 1868
    /*
     ***************************************************************************
     * Read machine uptime, independently of the number of processors.
     *
     * OUT:
     * @uptime  Uptime value in jiffies.
     ***************************************************************************
     */
    void read_uptime(unsigned long long *uptime)
    {
        FILE *fp; 
        char line[128];
        unsigned long up_sec, up_cent;
    
        if ((fp = fopen(UPTIME, "r")) == NULL)
            return;
    
        if (fgets(line, 128, fp) == NULL) {
            fclose(fp);
            return;
        }    
    
        sscanf(line, "%lu.%lu", &up_sec, &up_cent);
        *uptime = (unsigned long long) up_sec * HZ + 
                  (unsigned long long) up_cent * HZ / 100; 
    
        fclose(fp);
    
    }

按注释说明应该是返回的UPTIME与CPU数目无关，但是不太明白……

.. [#a] http://smilejay.com/2012/05/proc_uptime/
.. [#b] Documentation/filesystems/proc.txt - 1.8 
        Miscellaneous kernel statistics in /proc/stat
