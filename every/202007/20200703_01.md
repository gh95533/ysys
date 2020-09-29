[TOC]

# kettle 获取远程文件并下载



kettle remote file ftp download  step

前提：在本次TL项目建设中，需要的数据资源有一部分通过ftp的方式交换的。

1>获取远程更新或者新增的文件名

这个方案第一个难点就是如何获取新增的或者更改的文件名

1-1>执行脚本获取ftp中的文件名，新增或者修改

时间

1-2>与本地文件比对查看那些文件需要下载

2>从ftp中下载新增文件

这个方案第二个难点就是新增的文件可能多个需要将其全部下载完成

2-1>获取文件名并传参

2-2>设置循环下载

3>文件解析入库

这个方案第三个难点就是文本文件的分隔符

附件为远程ftp下载

![ftp_set_variable.ktr](../img_src/0dfefb224ec248689afea81f727bfcf3/attachment.png)

[ftp_set_variable.ktr](../img_src/fd8295eb28c84a528080d01b7bbd49e4/ftp_set_variable.ktr)

![hhht_ftp_copy_rows_to_result.ktr](../img_src/c04b2aeedf9c450d9107586a47a5844b/attachment.png)

[hhht_ftp_copy_rows_to_result.ktr](../img_src/e61846925c6348859feca9589b1ebdf7/hhht_ftp_copy_rows_to_result.ktf)

![hhht_ftp_result_variable_system_download.kjb](../img_src/4471f7a808fa405c829ca6b14f217b1a/attachment.png)
[hhht_ftp_result_variable_system_download.kjb](../img_src/5991b955eff64d14b20df5832e4de6bf/hhht_ftp_result_variable_system_download.kjb)

![hhht_ftp_system_download.kjb](../img_src/c0af2099c1024a239a2dfdae1973434f/attachment.png)
[hhht_ftp_system_download.kjb](../img_src/15b220077b1b406a9091a17d3eb31cb7/hhht_ftp_system_download.kjb)

![hhht_ftp_variable_system_download.kjb](../img_src/b9c77523b339486ab4e515818d99bfec/attachment.png)
[hhht_ftp_variable_system_download.kjb](../img_src/b8e5cdf4c1044258b3cb768ff85f1880/hhht_ftp_variable_system_download.kjb)

![SYSTEM_INFO.ktr](../img_src/3b5c8e6ece3c4dd2ac0b4d7140897ab3/attachment.png)
[SYSTEM_INFO.ktr](../img_src/fc1bf674349f424e87f60286e8d6ffc7/SYSTEM_INFO.ktr)