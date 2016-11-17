---
layout: post
title: awk&sed一些常用技巧
categories: [blog ]
tags: [test]
description: 包括随机数读文件等比较常用，但稍微有点复杂的技巧
---

# awk

## awk随机数


----------


    awk -F"\t" 'BEGIN{
    srand();}
    {
    value = int(rand()*100)
    }
    
    
**注意** srand()需要写在BEGIN模块，才能正常产生随机数，这是awk的工作机制决定的。
上述方法产生的随机数范围在 (0,100)
## awk调用shell
----------
1. getline
      通过在awk内使用管道,可以把shell命令的输出传送给awk
```awk
[zhinian.hwl]$ awk 'BEGIN{ "date" | getline date; print date; }'
Tue Dec  2 16:29:18 CST 2014
```

    顺便说一下getline的其他用法.
getline除了可以通过管道从shell命令里读取数据外,它还可以从标准输入(用"-"指定从标准输入读入,或者如果命令行没有任何输入文件且不用重定向符"<"指定文件,默认也是从标准输入读)和文件里读取数据;如果getline后面没有指定变量,则读取的数据会放到$0里面
```awk
 awk 'BEGIN{ getline < "sed.test"; print $0 }'
 ```
 
2. system
system的调用形式是system(cmd).system的返回值是cmd的退出状态.如果要获得cmd的输出,就要和getline结合
```awk
awk 'BEGIN{ while( system("ls -l") | getline line ){ print line } }'
```

这个部分还需要多加使用，我们也可以使用getline获得外部文件到数组， 详细在下面有标注



## awk数组
----------
一、定义方法
 
1：可以用数值作数组索引(下标)
>Tarray[1]=“cheng mo” Tarray[2]=“800927”

2：可以用字符串作数组索引(下标)
>Tarray[“first”]=“cheng ” Tarray[“last”]=”mo” Tarray[“birth”]=”800927”

使用中 print Tarray[1] 将得到”cheng mo” 而 print Tarray[2] 和 print[“birth”] 都将得到 ”800927” 。


##awk读取外部文件存到数组
--------
```awk
  1 #!/bin/awk
  2 BEGIN {
  3     count=1
  4     while (getline < nid_list) {
  5         nids[count] = $0
  6         count = count+1
  7     }
  8 }
```
外部调用的时候，使用：
>  awk -f tmp.awk -F "^A" -v nid_list=album_id_list  file_name
用-v 将 文件名传进去

#sed

##sed 修改匹配的第一个
----------------
在平常的修改中，有时候同一个匹配想可以匹配多次，但是我们只想修改匹配的第一个,举例来说
有一份文件匹配多个 keyword, 只修改第一个
```
sed -i '1,/keyword/{s/keyword/newkeyword/}' fileName
```


##sed的非贪婪匹配
--------
一般情况下， sed 的匹配都是 贪婪匹配 匹配最长， 但是 我们会经常遇到一些需要最短匹配的情况 。 距离来说 ， 一段 url :

```
http://xxx.com/luna?count=5&src=aitaobao&pidvid=&debug=true&q=%E6%89%8B%E6%9C%BA&srcpvid=1&srcip=1&navigator=true&acookie=abceds&itemid=43478031577,40542743067&nidquery=true

```

如果有一份 url组成的文件，需要改动每个url将里面的acookie=.*& 这个项删掉， 如果直接按照这个去做的话，acookie后面所有的项目都会被删掉，  这里 我们就需要去看一下 sed 的非贪婪匹配， 也就是 **最短匹配** 的正则：

```
/acookie=[^&]*&/
``` 

以上 用 acookie=[^&]*& 来代替 acookie=.*& 即可 。 目前原理还没有彻底弄清楚， 按照这个格式里面去写， 就可以。  

 
