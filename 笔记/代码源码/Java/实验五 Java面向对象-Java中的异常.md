### 第2关：捕获异常
```Java
package step2;  
  
import java.util.Scanner;  
  
public class Task {  
    public static void main(String[] args) {  
        Scanner sc = new Scanner(System.in);  
        int num1 = sc.nextInt();  
        int num2 = sc.nextInt();  
        /********* Begin *********/  
        if (num2 == 0) {  
            System.out.println("除数不能为0");  
        } else {  
            System.out.println(num1/num2);  
        }  
        /********* End *********/  
    }  
  
}
```

