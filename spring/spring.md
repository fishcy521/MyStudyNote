
spring 学习笔记
========

#一级标题
##二级标题
###三级标题
####四级标题
#####五级标题
######六级标题

这是一首简单的小情歌<br>
唱着人们心肠的曲折\<br>
我想我很快乐
当有你的温热
脚边的空气转了

		我是单行文本<br>

		看不穿 是你失落的魂魄
		猜不透是你瞳孔的颜色
		一阵风 一场梦 爱如生命般莫测

哪个人在天桥下留下等待工作的`电话号码`
[这是百度](www.baidu.com "我是百度")

* java
  * 流程控制
    * for
      * fori
      * 增强for循环
* c++
* c#
* c
* php

>中国
>阿萨德法师打发
>>山东
>>>济南
>>>>历下区



```html
<!doctype html>
<html lang="zh-cn">
  <head>
  </head>
  <body>
    <h1>
      hello world!
    </h1>
  </body>
</html>
```



```sql
select * from table_name
```



1. Spring基础配置
   * Spring框架四大原则
     1. 使用POJO进行轻量级和最小侵入式开发
     2. 通过依赖注入和基于接口编程实现松散耦合
     3. 通过AOP和默认习惯进行声明式编程
     4. 使用AOP和模板(template)减少模式化代码
2. 依赖注入
		Spring IoC容器(ApplicationContext)负责创建Bean，并通过容器将功能类Bean注入到你需要的Bean中。

	* 声明Bean的注解:
		- @Component组件,没有明确的角色
		- @Service在业务逻辑层(service层)使用
		- @Repository在数据访问层(dao层)使用
		- @Controller在展现层(MVC-->SpringMVC)使用
	* 注入Bean的注解,一般情况下通用
		- @Autowired: Srping提供的注解
		- @Inject: JSR-330提供的注解
		- @Resource: JSR-250提供的注解

		@Autowired、@Inject、@Resource可注解在set方法上或者属性上