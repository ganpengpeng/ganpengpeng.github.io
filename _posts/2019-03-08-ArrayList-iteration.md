---
layout: post
title:  "移除ArrayList元素使迭代器产生偏差"
categories: ArrayList
tags:  Java ArrayList
author: ganpeng
---

* content
{:toc}


看ArrayList相关解析时候发现一个小问题。  
ArrayList的迭代器实现为其内部私有类。其内部有一cursor变量记录下次应该返回的元素。而next()函数直接判断cursor是否等于Object数组size。
于是考虑这样的代码：



```java
List<String> list = new ArrayList<String>();
        //CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<String>();
        list.add("a");
        list.add("b");
        list.add("c");
        list.add("d");
        list.add("e");
        Iterator iterator = list.iterator();
        while(iterator.hasNext()){
            String str = (String) iterator.next();
            if(str.equals("d")){
                list.remove(str);
            }else{
                System.out.println(str);
            }
        }

        output：a b c
```

在上面代码中直接使用list.remove。于是数组中指定元素被删除，后面元素前移，size减一。

而cursor的值没有变化，也就是说本来应该在下一次next调用中返回的元素跑到了cursor索引前，于是上例中的e并未输出。

在这用情况下应该使用迭代器的remove方法，而不能使用List的remove方法。将list.remove改成iterator.remove即可输出**a b c e**。