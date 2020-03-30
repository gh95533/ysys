[TOC]

# kettle errors



kettle errors

其实这就是平时遇到的报错信息，如果是在安装或者调试某个过程中报错是，不会在这里重新记录一遍。

1、无法使用kitchen调用资源库作业

[root@qbname data-integration]# ./kitchen.sh /res:SPOON4 /user:admin /pass:admin /job:SJSC_JZKK_F_ZYK_T_TL_KKYPSJ

/opt/pdi-ce-5.2.0.0-209/data-integration

2017/05/25 09:06:14 - Kitchen - Start of run.

ERROR: Kitchen can't continue because the job couldn't be loaded.

检查相关信息

进入kettle软件的家目录（默认那个用户安装就是那个)，查看repository信息

cat /root/.kettle/repositories.xml

将kettle目录的所有.sh文件授予777权限

chmod 777 *.sh

检查调用sh文件信息，将路径添加上去,命令执行

说明一个问题就是可能环境变量哪里配置有问题

[root@qbname xingzhen]# cat /opt/pdi-ce-5.2.0.0-209/data-integration/sjsc_jzkk_f_zyk_t_tl_kkypsj.sh 

./kitchen.sh /rep:SPOON4 /user:admin /pass:admin /job:SJSC_JZKK_F_ZYK_T_TL_KKYPSJ

修改添加路径

[root@qbname xingzhen]# cat /opt/pdi-ce-5.2.0.0-209/data-integration/sjsc_jzkk_f_zyk_t_tl_kkypsj.sh 

cd /opt/pdi-ce-5.2.0.0-209/data-integration

./kitchen.sh /rep:SPOON4 /user:admin /pass:admin /job:SJSC_JZKK_F_ZYK_T_TL_KKYPSJ

2、linux环境下使用kettle的table input控件，无法在sql栏中点击，点击后整个程序退出

Xlib:  extension "RANDR" missing on display "localhost:10.0".

Attempting to load ESAPI.properties via file I/O.

Attempting to load ESAPI.properties as resource file via file I/O.

Not found in 'org.owasp.esapi.resources' directory or file not readable: /opt/data-integration/ESAPI.properties

Not found in SystemResource Directory/resourceDirectory: .esapi/ESAPI.properties

Not found in 'user.home' (/root) directory: /root/esapi/ESAPI.properties

Loading ESAPI.properties via file I/O failed. Exception was: java.io.FileNotFoundException

Attempting to load ESAPI.properties via the classpath.

SUCCESSFULLY LOADED ESAPI.properties via the CLASSPATH from '/ (root)' using current thread context class loader!

SecurityConfiguration for Validator.ConfigurationFile not found in ESAPI.properties. Using default: validation.properties

Attempting to load validation.properties via file I/O.

Attempting to load validation.properties as resource file via file I/O.

Not found in 'org.owasp.esapi.resources' directory or file not readable: /opt/data-integration/validation.properties

Not found in SystemResource Directory/resourceDirectory: .esapi/validation.properties

Not found in 'user.home' (/root) directory: /root/esapi/validation.properties

Loading validation.properties via file I/O failed.

Attempting to load validation.properties via the classpath.

validation.properties could not be loaded by any means. fail. Exception was: java.lang.IllegalArgumentException: Failed to load ESAPI.properties as a classloader resource.

SecurityConfiguration for Logger.LogServerIP not either "true" or "false" in ESAPI.properties. Using default: true

java: cairo-misc.c:380: _cairo_operator_bounded_by_source: Assertion `NOT_REACHED' failed.

./spoon.sh: line 200:  2068 Aborted                 (core dumped) "$_PENTAHO_JAVA" $OPT $STARTUP -lib $LIBPATH "${1+$@}"

报错信息比较长，但是我们只需要关注最后两行，前面是kettle自己的bug;暂时没有解决方案，"java: cairo-misc.c:380: _cairo_operator_bounded_by_source: Assertion `NOT_REACHED' failed.

./spoon.sh: line 200:  2068 Aborted                 (core dumped) "$_PENTAHO_JAVA" $OPT $STARTUP -lib $LIBPATH "${1+$@}""

我们需要在opt="..."

添加一行即可

-Dorg.eclipse.swt.internal.gtk.cairoGraphics=false

整个OPT设置

OPT="$OPT $PENTAHO_DI_JAVA_OPTIONS -Djava.library.path=$LIBPATH -DKETTLE_HOME=$KETTLE_HOME -DKETTLE_REPOSITORY=$KETTLE_REPOSITORY -DKETTLE_USER=$KETTLE_USER -DKETTLE_PASSWORD=$KETTLE_PASSWORD -DKETTLE_PLUGIN_PACKAGES=$KETTLE_PLUGIN_PACKAGES -DKETTLE_LOG_SIZE_LIMIT=$KETTLE_LOG_SIZE_LIMIT -DKETTLE_JNDI_ROOT=$KETTLE_JNDI_ROOT -Dorg.eclipse.swt.internal.gtk.cairoGraphics=false"

文件执行

其他的报警信息不需要太多关注;其实其他的报错信息在centos每个版本几乎都有，不会太影响

3、生产环境下gpload控件更新时，无法选择YES

kettle greenplum loader match not Y

在生产环境（5.2,5.4）中发现如何怎么修改kettle greenplum loader match not Y，不知道到底是因为什么，导致我的生产环境,每次选择完Y,保存都会变为N；实在没有办法，只能到数据库修改脚本。

先到数据库找到表：r_transfromation;找到方案的id_transfromation后；找到表r_step;通过id_transformation找到id_step;找到表r_step_attribute后按照自己的想法修改。

