## 集合判空
使用spring包下的
```
CollectionUtils.isEmpty(list)
```
> 比list.isEmpty()多个对list==null的判断

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
