```java
package case6;  
public class finalTest {  
    public static void main(String args[]) {  
        Bike1 obj = new Bike1();  
        obj.run();  
        Honda honda = new Honda();  
        honda.run();  
        Yamaha yamaha = new Yamaha();  
        yamaha.run();  
    }  
}  
//不可以修改 final 变量的值  
// final方法,不可以重写  
//// 不可以扩展 final 类  
//请在此添加你的代码  
/********** Begin *********/  
class Bike1 {  
    int speedlimit = 90;// final variable 将final删掉  
    void run() {  
        speedlimit = 120; // 不可以修改 final 变量的值  
        System.out.println("speedlimit=120");  
    }  
}  
class Bike2 {  
    void run() {// 将final删掉  
        System.out.println("running");  
    }  
}  
class Honda extends Bike2 {  
    void run() { // final方法,不可以重写  
        System.out.println("running safely with 100kmph");  
    }  
}  
class Bike3 { // 将final删掉  
}  
class Yamaha extends Bike3 { // 不可以扩展 final 类  
    void run() {  
        System.out.println("running safely with 100kmph");  
    }  
}  
/********** End **********/
```