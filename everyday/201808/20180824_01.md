[TOC]

# pg 去除字段特殊字符

​	特殊字符指的就是标点符号，如字段`abc过后123，估计？。多发几款；戴假发；`这些字段中就要将这些字段给他注销掉

```
 select regexp_replace('过会#ghhhh过会drdf#$%@&*($%6','[[:punct:]]+',' ','g') ;
    regexp_replace    
----------------------
 过会 ghhhh过会drdf 6
```

