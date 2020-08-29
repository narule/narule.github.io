jq 



linux json解析器



```shell
cat blog.json | jq '.note | has("IO")'  
# 语句含义
# 1.读取blog.json 文件  
# 2.内容用 jq 解析
# 3.查找到 key为"note" 的数据 
# 4.查看是否包含 "key 为 IO的数据"
```



```json
{
    "note":{
        "java":{
            "IO":{
                IO_html:{
                    "title":"IO",
                    "create_time":"2020-06",
                    "update_time":"2020-07"
                }
            }
        }
    }
}
//folder note/java/IO
//file  note/java/IO/IO.html
```

how to read note.java.IO  ? 

and add 

```json
netty_html:{
    "title":"netty",
    "create_time":"2020-07",
    "update_time":"2020-07"
}
```

`cat blog.json | jq -S '.note | map(. + {insertJs:{"sd":"dsa","ss":"0"}})'`  这个没有键

`cat blog.json | jq '.note  | (. +  {insertJs:{"sd":"dsa","ss":"0"}})'`  添加

`cat blog.json | jq '.note |= del(.IO)'`   删除

`cat blog.json | jq '.note.IO  |=  .+  {insertJs:{"sd":"dsa","ss":"0"}}'` 添加然后合并

`cat blog.json | jq '.note.IO  |=  .+  {insertJs:{"sd":"dsa","ss":"0"}}'` 

 `cat blog.json | jq '.about_html.title = "11111"'` 修改值

` cat blog.json | jq '.note.IO.Tread =  {nio:{},netty:{}}'` 等效 `cat blog.json | jq '.note.IO.Tread |= .+ {nio:{},netty:{}}'`  

创建一个脚本方法，如果没有key ,创建一个







```shell
dir="note/javaIO"
keyst="${$dir////.}"   # note/java/IO  -> note.java.IO

fnc checkkey(key){
key_ = note;
subkey = java.io
if(!jq '. | has("${key_}")'  blog.json){
	create {"note":{}}
	}
}
if(!z subkey){
	checkkey(subkey)
}

if(!jq '.note | has("IO")'){
	
} 
```



### 将文件内容修改重写如文件 

名字不能为源文件名，文件会被置空。

可以这样：

```shell
cat blog.json | jq '.note.IO  |=  .+  {insertJs:{"sd":"dsa","ss":"0"}}' > temp 
cat temp > blog.json
## 必须分两次命令，不然依旧是空值

```

