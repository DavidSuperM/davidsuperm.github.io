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
LocalDateTime startTime = LocalDateTime.of(LocalDate.now(), LocalTime.MIN); //当天零点
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

// jsonstring转对象
User user = mapper.readValue(json, User.class);
//jsonstring转json对象
JsonNode jsonNode = mapper.readTree(json);
JsonNode j2 = jsonNode.get("f2");
String f2Str = jsonNode.get("f2").asText();
double f2Dbl = jsonNode.get("f2").asDouble();
int    f2Int = jsonNode.get("f2").asInt();
long   f2Lng = jsonNode.get("f2").asLong();
ObjectNode objectNode = (ObjectNode) jsonNode;
objectNode.put(key,value);
//对象转jsonstring
String result = mapper.writeValueAsString(value);
```
