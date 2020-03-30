[TOC]

# kettle 判断 效率慢 策略



&ensp;&ensp;&ensp;&ensp;在之前数据抽取流程慢的方案中，发现不是文件解析慢，而是数据入库慢，那么如何提高数据入库速度就是我们后面需要解决的问题。

### location step slow

&ensp;&ensp;&ensp;&ensp;如何确定那个步骤慢呢，目前经常使用一个控件(nothing),这个控件是什么也不做，但是却可以明显测试出速率如何。如前者csv读取特别慢，将csv与oracle中间的数据转换修改成nothing 控件，查看csv读取速度，相反就是oracle插入特别慢。