# 模板
模板系统遇到变量名中的点号时会按照下述顺序尝试查找：
- 字典查找（如 foo["bar"]）
- 属性查找（如 foo.bar）
- 方法调用（如 foo.bar()）
- 列表索引查找（如 foo[2]）


# 数据库

如果想让日期字段（如 DateField、TimeField、DateTimeField）或数值字段（如 IntegerField、DecimalField、FloatField）接受空值，要同时添加 null=True 和 blank=True。


# 后端 admin
filter_horizontal 精致的选择框

类视图

自定义标签 过滤器