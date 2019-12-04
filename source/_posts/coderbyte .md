---
title: coderbyte练习题--匹配字符串的特殊字符
category: 算法
tags:
  - coderbyte
  - java
abbrlink: 1fda1044
date: 2017-10-25 00:00:00
---

## 字符串中两个数字之间 有三个‘？’返回 true

```
import java.util.*;
import java.io.*;

class Main {  
  public static String QuestionsMarks(String str) {

    // code goes here   
    /* Note: In Java the return type of a function and the
       parameter types being passed are defined, so this return
       call must match the return type of the function.
       You are free to modify the return type. */
        int len =str.length();
       boolean isFlag=false;
       for(int i=0;i<len;++i){
           char c = str.charAt(i);
           int n =(int)c;
           if(n >= 48 && n<=57){
               int s = n-48;
                for(int j=i+1;j<len;++j){
                        char c1= str.charAt(j);
                        int n1 = (int)c1;
                        if(n1>=48 && n1<=57){
                            int s1 = n1-48;
                            if(s1 + s == 10){
                                String v = str.substring(i+1,j);
                        //        System.out.print(v+" ");
                                int count =0 ;
                                for(int k=0;k<v.length();++k){
                                    if(v.charAt(k) == '?'){
                                        count++;
                                    }
                                }
                                if(count==3){
                           //         System.out.print(" && ");
                                   isFlag=true;
                                }else {
                             //      System.out.print(" @@ ");
                                    isFlag=false;
                                    return "false";
                                }
                                j=len;
                            }
                        }
                }
       }
       }
       if(isFlag)
       return "true";
    return "false";
  }
  public static void main (String[] args) {  
    // keep this function call here     
    Scanner s = new Scanner(System.in);
    System.out.print(QuestionsMarks(s.nextLine()));
  }   
}
```
