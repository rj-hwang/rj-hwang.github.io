# maven test 运行特定单元测试类的方法

## 1. 运行特定的测试类：

```
mvn test -D test=$ClassName
```

`$ClassName` 为要运行的测试类的类名（不要带扩展名），多个测试类间用英文逗号连接即可，类名支持 `*` 通配符，范例：
- mvn test -D test=MyClassTest
- mvn test -D test=MyClass1Test,MyClass2Test
- mvn test -D test=MyClass*Test,YourClassTest
- mvn test -D test=MyClassTest

## 2. 运行测试类中指定的方法：

```
mvn test -D test=$ClassName#$MethodName
```

`$MethodName` 为要运行的单元测试方法的名称，与类名之间用 `#` 分隔开，支持 `*` 通配符，范例：

- mvn test -D test=MyClassTest#test1
- mvn test -D test=MyClassTest#*test*

> 需要 maven-surefire-plugin:2.7.3 以上版本才支持


[163 旧址]: http://rongjih.blog.163.com/blog/static/335744612010102911363452/