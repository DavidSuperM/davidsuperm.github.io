# java问题合集
1.  
问题：报错:lineNumber: 1; columnNumber: 1225; 元素内容必须由格式正确的字符数据或标记组成。
原因： mybatis配置文件中或者<script>中不能出现小于号 即 < 这个符号
解决：逻辑改成 > 或者 < 用 &lt; 代替