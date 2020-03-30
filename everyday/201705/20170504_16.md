[TOC]

# kettle copy lot's table



xj现场2000多代码表，不可能一个一个的拷贝，发现表码是两个标准字段，那么先将表结构创建出来，然后使用kettle将其全部抽取过来

下面是我测试的脚本，可以直接使用，只需要修改两个脚本，分别是oracle_copy.ktr修改数据库链接，表名；oracle_ins.ktr修改数据库链接

[get_result_set_variable.ktr](../img_src/da48d6a39470413f92b310f693395456/get_result_set_variable.ktr)

[get_result_set_variable_job.kjb](../img_src/30d22fb0580e4a7397115480ba4596f7/get_result_set_variable_job.kjb)

[oracle_copy.ktr](../img_src/ff68020a94064d08a382c882d4048dbe/oracle_copy.ktr)

[oracle_copy_job.kjb](../img_src/7681eb66857648bea36321a14314a1d4/oracle_copy_job.kjb)

[oracle_ins.ktr](../img_src/6f7068b136eb4ea7be76ad6ca1ea6dfa/oracle_ins.ktr)

[oracle_ins_job.kjb](../img_src/671c393febcf4340a9dafa15363db2d9/oracle_ins_job.kjb)

此次脚本命令和华为mppdb对接是，经常会出现运行一段时间就不会再跑了，这个问题需要排查一下到底是什么问题，不过脚本就是这个样子了，后期再修改