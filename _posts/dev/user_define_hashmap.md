date: 2015-2-13
title: 使用自定义类型作为HashMap接口的键或值
tags: hashmap
category: java
---

```java
// 注意：如果使用了非系统类型作为key,则必须覆写equals()和hashCode()方法，否则无效。
// 使用自定义的类型作为键
import java.util.Map ;
import java.util.HashMap ;
class Person{
    private String name ;
    private int age ;
    public Person(String name,int age){
        this.name = name ;
        this.age = age ;
    }
    public String toString(){
        return "姓名：" + this.name + "；年龄：" + this.age ;
    }
    public boolean equals(Object obj){ //覆写
        if(this==obj){
            return true ;
        }
        if(!(obj instanceof Person)){
            return false ;
        }
        Person p = (Person)obj ; //向下转型
        if(this.name.equals(p.name)&&this.age==p.age){
            return true ;
        }else{
            return false ;
        }
    }
    public int hashCode(){  //覆写，实现编码唯一
        return this.name.hashCode() * this.age ;
    }
};
public class HashMapDemo06{
    public static void main(String args[]){
        Map<String,Person> map = null ;
        map = new HashMap<String,Person>() ;
        map.put("zhangsan",new Person("张三",30)) ; //增加内容
        System.out.println(map.get("zhangsan")) ;
 
        Map<Person,String> map1 = null ;
        map1 = new HashMap<Person,String>() ;
        Person per = new Person("张三",30) ;
        map1.put(per,"zhangsan") ; //增加内容，使用了per对象
        System.out.println(map1.get(per)) ; //注意是per对象
        Map<Person,String> map2 = null ;
        map2 = new HashMap<Person,String>() ;
        map2.put(new Person("张三",30),"zhangsan") ; //增加内容，使用了匿名对象
        System.out.println(map2.get(new Person("张三",30))) ; //注意是匿名对象
    }
};
```
