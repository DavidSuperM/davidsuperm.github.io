## 概念
Java 序列化技术可以使你将一个对象的状态写入一个Byte 流里（系列化），并且可以从其它地方把该Byte流里的数据读出来（反序列化）。

## 使用
类实现接口
```
@Data
class Person implements Serializable{	
	private static final long serialVersionUID = 1L; 
	String name;
	int age;	
}
```
接受方要设置同样的serialVersionUID才可以正确的将Byte流数据转为对象。
如果没有特殊需求的话，使用用默认的 1L 就可以。
随机生成的序列化ID作用：有些时候，通过改变序列化 ID 可以用来限制某些用户的使用。

## 使用场景
1. 想把对象通过网络进行传播的时候
2. 想把的内存中的对象状态保存到一个文件中或者数据库中时候

## Questions
1. springboot返回给前端的对象为什么不序列化也能传输？
答：springboot中返回给前端的对象不需要序列化，因为springboot框架会将对象转换成json字符串传给前端，而字符串是不需要序列化的

2. entity类为什么不序列化也能存储到mysql中
答：因为entity类的属性类型，Long，String等等都已经实现了Serializable。
但是最好entity还是实现Serializable，因为当存储nosql时，数据库中没有对应的数据类型时，或者需要将entity对象传输的时候，就需要实现Serializable。