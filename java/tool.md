## 1.得到指定date  (现在不推荐用Date，改用LocalDateTime)
```
//得到指定日期的0时0分0秒
Date currentDate = new Date();
Calendar calendar = Calendar.getInstance();
calendar.setTime(currentDate);
calendar.set(Calendar.HOUR_OF_DAY, 0);
calendar.set(Calendar.MINUTE, 0);
calendar.set(Calendar.SECOND, 0);
Date zero = calendar.getTime();

//指定一天前的时间
calendar.add(Calendar.DAY_OF_MONTH, -1);

// string转date
SimpleDateFormat sdf =  new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date date = sdf.parse("2008-07-10 19:20:00"); 

// date转string
SimpleDateFormat sdf =  new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String str = sdf.format(new Date()); 

// 得到时间戳
Date date = new Date();
long m = date.getTime();  
// 得到两个时间的秒数差
Date date1 = new Date();
Date date2 = new Date();
long m1 = date1.getTime(); 
long m2 = date2.getTime(); 
int diff = (m2 - m1)/1000
```

## LocalDateTime
问：为什么用LocalDateTime不用Date
答：因为Date要用SimpleDateFormat格式化时间，但SimpleDateFormat是线程不安全的。详见<https://zhuanlan.zhihu.com/p/87555377>

1. 获取当前时间
```
LocalDateTime localDateTime = LocalDateTime.now()
```
2. 获取当前日期
```
LocalDate localDate = LocalDate.now();
```
3. 时间加减
```
localDateTime.minusDays(1);   // 前一天
localDateTime.plusDays(1);    // 后一天
```
4. 获取时间戳
```
//获取秒数
Long second = LocalDateTime.now().toEpochSecond(ZoneOffset.of("+8"));
//获取毫秒数
Long milliSecond = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();
```
5. 时间字符串转换
```
// LocalDateTime转为String
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String str = LocalDateTime.now().format(formatter);
// String转LocalDateTime
LocalDateTime localDateTime = LocalDateTime.parse("2018-07-28 14:11:15",  DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

6. 获取指定时间
```
LocalDateTime startTime = LocalDateTime.of(LocalDate.now(), LocalTime.MIN); //当天零点,当天0点
LocalDateTime startTime = LocalDateTime.now().withHour(0).withMinute(0).withSecond(0).withNano(0); //当天零点
LocalDateTime startTime = LocalDateTime.of(2020,12,12,0,0,0,0); //当天零点
LocalDateTime endTime = LocalDateTime.of(LocalDate.now(), LocalTime.MAX);  //当天结束时间 
LocalDateTime endTime = LocalDateTime.now().withHour(23).withMinute(59).withSecond(59).withNano(0);   //当天结束时间 其实withNano应该填很多9，但是因为mysql里timestamp精度只存到秒，所以毫秒设为0也可以
LocalDateTime endTime = LocalDateTime.of(2020,12,12,23,59,59,0); //当天结束时间
```


## json转换
用jsckson(fastjson有漏洞)
```
// 引入依赖
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.10.6</version>
</dependency>

private ObjectMapper mapper = new ObjectMapper();
// json字符串转对象
User user = mapper.readValue(json, User.class);
// 对象转json字符串
String result = mapper.writeValueAsString(value);
// json字符串转json对象
JsonNode jsonNode = mapper.readTree(json);
JsonNode j2 = jsonNode.get("f2");
String f2Str = jsonNode.get("f2").asText();
double f2Dbl = jsonNode.get("f2").asDouble();
int    f2Int = jsonNode.get("f2").asInt();
long   f2Lng = jsonNode.get("f2").asLong();
ObjectNode objectNode = (ObjectNode) jsonNode;
objectNode.put(key,value);
// json对象转json字符串
String res = jsonNode.toString();
// jackson构建json对象
ObjectNode objectNode = mapper.createObjectNode();
objectNode.put("id","3");
System.out.println(objectNode.toString());
// jackson构建json数组
ObjectNode objectNode = mapper.createObjectNode();
ArrayNode arrayNode = objectNode.putArray("arr1");
arrayNode.add("3");
arrayNode.add("4");
System.out.println(arrayNode.toString());
```

## java8 stream使用
.stream可以将集合内的元素做转换
.foreach不可用对原集合元素转换，但是可以操作取出的每个元素(如果取出的是对象，也可以改变对象的属性)
```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).collect(Collectors.toList());
 
List<Integer> squaresList = numbers.stream().map( i -> {
            i = i*i;
            return i;
        }).collect(Collectors.toList());

numbers.forEach(i-> System.out.println(i));
```
参考：<https://blog.csdn.net/u014042066/article/details/76372116>

## UUID
### 定义
UUID(Universally Unique Identifier)全局唯一标识符,是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。
UUID是一个128位长的数字，一般用16进制表示。算法的核心思想是结合机器的网卡、当地时间、一个随即数来生成UUID。从理论上讲，如果一台机器每秒产生10000000个UUID，则可以保证（概率意义上）3240年不重复。
### 使用
```
String uuid = UUID.randomUUID().toString();

// 结果
d0cd48ef-313a-405c-ac76-12cce5657dca
d7e1e24b-9574-4ea0-a08f-4cfa6965035f
c6b75eef-f6ca-4963-8c4f-26b72cb15eee
```


## @RequestBody和@RequestParam区别
get请求的参数或者post请求Content-Type为application/x-www-form-urlencoded时，后端用@RequestParam 接收
```
public String insert(@RequestParam("name") String name, @RequestParam("age") Integer age) {
    return ""
}
```
post请求Content-Type为application/json时，后端用 @RequestBody 接收
前端传的是字符串，后端springboot会自动转换成对象,后端返回对象也会将对象序列化成字符串
```
public String insert(@RequestBody("user") User user) {
    return ""
}
```
参考:<https://cloud.tencent.com/developer/article/1414464>

## 浅拷贝与深拷贝（深度拷贝，深复制，深度复制，深度克隆，深克隆）
浅拷贝：拷贝对象的基本类型属性，对象类型属性拷贝的是引用
方法: 实现Cloneable接口，复写clonen函数，函数里调用super.clone, main里直接对对象调用 object.clone即可
```
public class Student implements Cloneable { 
    // 对象引用 
   private Subject subj; 
   private String name; 
    ......
 public Object clone() { 
      //浅拷贝 
      try { 
         // 直接调用父类的clone()方法
         return super.clone(); 
      } catch (CloneNotSupportedException e) { 
         return null; 
      } 
   } 
}
```
深拷贝：对象属性也生成新的对象拷贝
方法：用jackson，需要先引入jar包，类要用无参构造函数
```
Address address = new Address("杭州", "中国");
User user = new User("大山", address);
// 使用Jackson序列化进行深拷贝
ObjectMapper objectMapper = new ObjectMapper();
User copyUser = objectMapper.readValue(objectMapper.writeValueAsString(user), User.class);
```

## 获取随机数，随机字符串
```
// 使用指定的字符生成5位长度的随机字符串  
r = RandomStringUtils.random(5, new char[] { 'a', 'b', 'c', 'd', 'e',  'f', '1', '2', '3' });  
// 生成指定长度的字母和数字的随机组合字符串  
r = RandomStringUtils.randomAlphanumeric(5);  
// 生成随机数字字符串  
r = RandomStringUtils.randomNumeric(5);   
// 生成随机[a-z]字符串，包含大小写  
r = RandomStringUtils.randomAlphabetic(5);  
// 生成从ASCII 32到126组成的随机字符串  
r = RandomStringUtils.randomAscii(4);  
```