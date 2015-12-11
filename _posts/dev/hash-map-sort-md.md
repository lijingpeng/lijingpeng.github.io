title: HashMap的两种排序方式
date: 2015-05-03 12:57:56
tags:
category: Java
---

```java
Map<String, Integer> map = new HashMap<String, Integer>();
map.put(“d”, 2);
map.put(“c”, 1);
map.put(“b”, 1);
map.put(“a”, 3);

List<Map.Entry<String, Integer>> infoIds =
new ArrayList<Map.Entry<String, Integer>>(map.entrySet());

//排序前
for (int i = 0; i < infoIds.size(); i++) {
String id = infoIds.get(i).toString();
System.out.println(id);
}

//d 2
//c 1
//b 1
//a 3

//排序
Collections.sort(infoIds, new Comparator<Map.Entry<String, Integer>>() {
public int compare(Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) {
//return (o2.getValue() – o1.getValue());
return (o1.getKey()).toString().compareTo(o2.getKey());
}
});

//排序后
for (int i = 0; i < infoIds.size(); i++) {
String id = infoIds.get(i).toString();
System.out.println(id);
}

//根据key排序
//a 3
//b 1
//c 1
//d 2
//根据value排序
//a 3
//d 2
//b 1
//c 1
```
