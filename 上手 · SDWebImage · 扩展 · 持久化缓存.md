# 上手 · SDWebImage · 扩展 · 持久化缓存

- Plist 属性列表文件
- NSUserdefaluts 偏好设置
- NSKeyedArchiver 归档 解档
- FMDB 数据库
- CoreData



Plist 是轻量级明文存储方式，一般都是 XML 格式，可以存储少量切不重要的数据：

城市列表数据

中国邮政编码

中国银行列表等通用数据



NSUserdefaluts 可以存储一些简单的数据，一般都是一些应用的配置

NSKeyedArchiver 储存一些复杂的数据 



NSKeyedArchiver 存储的对象需要实现 NSCoding 协议并且实现了当中的两个协议方法



沙盒介绍

Bundle 跟 沙盒 之间的区别



Document

Library （Library/Caches，Library/Preference）

temp



