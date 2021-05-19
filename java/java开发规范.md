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

## 浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals来判断。
```
float a = 1.0F - 0.9F;
float b = 0.9F - 0.8F;
if (a == b) {
// 预期进入此代码块，执行其它业务逻辑
// 但事实上 a==b 的结果为 false
}
Float x = Float.valueOf(a);
Float y = Float.valueOf(b);
if (x.equals(y)) {
// 预期进入此代码块，执行其它业务逻辑
// 但事实上 equals 的结果为 false
}
```
解决方法：
```
(1) 指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。
float a = 1.0F - 0.9F;
float b = 0.9F - 0.8F;
float diff = 1e-6F;
if (Math.abs(a - b) < diff) {
System.out.println("true");
}
(2) 使用 BigDecimal 来定义值，再进行浮点数的运算操作。
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);
// 较应使用 compareTo()方法，而不是 equals()方法，equals()方法会比较值和精度（1.0 与 1.00 返回结果为 false），而 compareTo()则会忽略精度。
if (x.compareTo(y) == 0) {
System.out.println("true");
}
```

## java开发手册
【强制】所有的 POJO 类属性必须使用包装数据类型。
【强制】RPC 方法的返回值和参数必须使用包装数据类型。
【推荐】所有的局部变量使用基本数据类型。
【强制】只要覆写 equals，就必须覆写 hashCode。

## 【强制】在使用 Collection 接口任何实现类的 addAll()方法时，都要对输入的集合参数进行NPE 判断。
说明：在 ArrayList#addAll 方法的第一行代码即 Object[] a = c.toArray(); 其中 c 为输入集合参数，如果
为 null，则直接抛出异常。

## Map遍历
```
Map<String, String> map = new HashMap<>(16);
map.put("1","a");
for(Map.Entry<String, String> m :map.entrySet()){
    m.getKey();
    m.getValue();
}
// jdk8及以后推荐下面
map.forEach((key,value)->{
    System.out.println(key);
    System.out.println(value);
});
```

## 【强制】对于需要使用超大整数的场景，服务端一律使用 String 字符串类型返回，禁止使用Long 类型。
说明：Java 服务端如果直接返回 Long 整型数据给前端，JS 会自动转换为 Number 类型（注：此类型为双
精度浮点数，表示原理与取值范围等同于 Java 中的 Double）。Long 类型能表示的最大值是 2 的 63 次方
-1，在取值范围之内，超过 2 的 53 次方 (9007199254740992)的数值转化为 JS 的 Number 时，有些数
值会有精度损失。扩展说明，在 Long 取值范围内，任何 2 的指数次整数都是绝对不会存在精度损失的，所
以说精度损失是一个概率问题。若浮点数尾数位与指数位空间不限，则可以精确表示任何整数，但很不幸，
双精度浮点数的尾数位只有 52 位。
反例：通常在订单号或交易号大于等于 16 位，大概率会出现前后端单据不一致的情况，比如，"orderId": 
362909601374617692，前端拿到的值却是: 362909601374617660。

## 【强制】在翻页场景中，用户输入参数的小于 1，则前端返回第一页参数给后端；后端发现用户输入的参数大于总页数，直接返回最后一页。

# Mysql规范
## 【强制】小数类型为 decimal，禁止使用 float 和 double。
说明：在存储的时候，float 和 double 都存在精度损失的问题，很可能在比较值的时候，得到不正确的
结果。如果存储的数据范围超过 decimal 的范围，建议将数据拆成整数和小数并分开存储。

## 【推荐】单表行数超过 500 万行或者单表容量超过 2GB，才推荐进行分库分表。
说明：如果预计三年后的数据量根本达不到这个级别，请不要在创建表时就分库分表。

## 【强制】在 varchar 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度。

##7. 【推荐】利用延迟关联或者子查询优化超多分页场景。
说明：MySQL 并不是跳过 offset 行，而是取 offset+N 行，然后返回放弃前 offset 行，返回 N 行，那当
offset 特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行 SQL
改写。
正例：先快速定位需要获取的 id 段，然后再关联：
```
SELECT t1.* FROM 表 1 as t1, (select id from 表 1 where 条件 LIMIT 100000,20 ) as t2 where t1.id=t2.id
```




