# java问题合集
## 1. mybatis不能用<符号
问题：报错:lineNumber: 1; columnNumber: 1225; 元素内容必须由格式正确的字符数据或标记组成。
原因： mybatis配置文件中或者<script>中不能出现小于号 即 < 这个符号
解决：逻辑改成 > 或者 < 用 &lt; 代替

## 2. springboot定时任务不推荐启动时就执行
在定时任务的方法上加
@PostConstruct
定时任务方法会在启动应用时执行一次，但是此时bean还没加载好，获取bean是获取不到的

