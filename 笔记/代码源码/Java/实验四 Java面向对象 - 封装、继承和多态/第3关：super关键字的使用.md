```java
package case3;  
  
public class case3 {  
    public static void main(String[] args) {  
        // 实例化一个Student类的对象s，为Student对象s中的school赋值，打印输出信息  
        /********* begin *********/  
        Student zhangsan = new Student();  
        zhangsan.name = "张三";  
        zhangsan.age = 18;  
        zhangsan.school = "哈佛大学";  
  
        zhangsan.printInfo();  
        /********* end *********/  
    }  
}  
  
class Person {  
    /********* begin *********/  
    String name;  
    int age;  
  
    void printInfo(){  
        System.out.print("姓名：" + name + "，年龄：" + age);  
    }  
    /********* end *********/  
}  
  
class Student extends Person {  
    /********* begin *********/  
    String school;  
  
    void printInfo(){  
        super.printInfo();  
        System.out.print("，学校：" + school);  
    }  
    /********* end *********/  
}
```
