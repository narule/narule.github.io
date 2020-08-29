

### HTTPS 证书

https://certbot.eff.org/



### 定时任务

cron

crontab 定时任务表



### gizp 压缩图片 无用



### magick 图片工具

#### 安装 

参考https://www.cnblogs.com/ifme/p/12193591.html

```bash
yum install ImageMagick
```



#### 使用

转换

`convert input.png output.jpg`

修改大小

`convert input.png -resize  50% output.png`

`convert input.png -resize  100x100 output.png`

批量转换



```shell
for file in *.png; do convert $file ${file%%.*}.jpg; done

# 批量修改大小
for file in *.jpg; do convert $file -resize ${file%%.*}.jpg; done
```

