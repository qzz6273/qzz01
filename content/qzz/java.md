1.对于abstract类， 不能使用new运算符创建该类的对象。
2.匿名内部类没有类名，不能自定义构造方法（![[Pasted image 20261118193339.png]]）
3.内部类4中修饰都能；外部类只能用public和defalut
`class`：Java 用来定义**外部类**的关键字
4.编译器只检查语法是否合规
5.- `throws`：**声明异常**（写在方法头，告诉别人我可能出异常）
- `throw`：**手动抛出异常对象**（写在方法体内，真正抛出）![[Pasted image 20261118194340.png]]
6.// 这不是构造！有void，它就是普通方法
`public void Person(){ }`

// 这是main主方法，程序入口，和构造没关系
`public static void main(String[] args) { }`
7.toString有返回值
```
public String toString() {
    return "我是返回的字符串";
}
```
```
// 构造方法（无返回）
public Person(){}

// toString（有返回，返回String）
public String toString(){}

// void普通方法（无返回）
public void test(){}
```

8.Exception：单纯所有**异常类**的父类（不含 Error）