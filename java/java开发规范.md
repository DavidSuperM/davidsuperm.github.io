## 集合判空
使用spring包下的 org.springframework.util.CollectionUtils; （我觉得用apache包下的也可以）
```
CollectionUtils.isEmpty(list)
```
> 比list.isEmpty()多个对list==null的判断

## 字符串判空
使用 org.apache.commons.lang3.StringUtils; 报下的
```
StringUtils.isEmpty();
```

## 字符串转整型浮点型
```
String str = "12";
long a = NumberUtils.toLong(str);
```
> 比Long.parseLong(str)多个对str==null的判断

## 基本数据类型转String
```
int a = 5;
String s = Integer.toString(a);
```

## 所有方法使用了泛型的必须要显示声明出来，没有设置泛型值也要加<Void>

## hashmap初始化容量
initialCapacity = (需要存储的元素个数 / 负载因子) + 1
initialCapacity = (需要存储的元素个数 / 0.75) + 1
这种算法实际上是一种使用内存换取性能的做法，在真正的应用场景中要考虑到内存的影响。

## ArrayList初始化容量
默认是10，满了则扩容成1.5倍，如果知道要存的数据量大小，最好指定值

## mybatis insertList要判断空或者size是否等于0，否则运行会报错