## 1.得到指定date
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
