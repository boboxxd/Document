# ubuntu 修改时区及时间

1.首先查看时区：

```
swfsadmin @ swfsubuntu：〜$ date  - R
星期二，17年12月2013年 18：23：01 + 0800
```

如果要修改时区，执行sudo tzselect

2.选择区域：亚洲

```
swfsadmin @ swfsubuntu：〜$ sudo tzselect
[须藤]密码为swfsadmin：
对不起，再试一次。
[须藤]密码为swfsadmin：
请确定一个位置，以便正确设置时区规则。
请选择大陆或海洋。
 1 ）非洲
  2 ）美洲
  3 ）南极
  4 ）北冰洋
  5 ）亚洲
  6 ）大西洋
  7 ）澳大利亚
  8 ）欧洲
  9 ）印度洋
 10 ）太平洋
 11）无 - 我想用Posix TZ格式指定时区。
＃？五
```

3.选择国家：中国

```
请选择一个国家
 1）阿富汗            18）以色列                 35 ）巴勒斯坦
  2）亚美尼亚                19）日本                  36 ）菲律宾
  3）阿塞拜疆             20）约旦                 37 ）卡塔尔
  4）巴林                21）哈萨克斯坦             38 ）俄罗斯
  5）孟加拉国             22）韩国（北）          39 ）沙特阿拉伯
  6不丹                 23）韩国（南）          40 ）新加坡
  7）文莱                 24）科威特                 41 ）斯里兰卡
  8）柬埔寨               25）吉尔吉斯斯坦             42 ）叙利亚
  9）中国                  26）老挝                   43 ）台湾
 10）塞浦路斯                 27）黎巴嫩                44 ）塔吉克斯坦
 11）东帝汶             28）澳门                  45 ）泰国
 12）格鲁吉亚                29）马来西亚               46 ）土库曼斯坦
 13）香港              30）蒙古               47 ）阿拉伯联合酋长国
 14）印度                  31）缅甸（缅甸）        48 ）乌兹别克斯坦
 15）印度尼西亚              32）尼泊尔                  49 ）越南
 16）伊朗                   33）阿曼                   50 ）也门
 17）伊拉克                   34 ）巴基斯坦
＃？9
```

4.选择时区：北京时间

```
请选择以下时区之一。
1）华东 - 北京，广东，上海等
 2 ）黑龙江（除Mohe），吉林
 3）中国中部 - 四川，云南，广西，陕西，贵州等
 4）西藏和新疆
大部分5）西藏西藏＆新疆
＃？1
```

5.确认验证：

```
提供以下信息：

        中国
        华东 - 北京，广东，上海等

所以TZ = ' 亚洲/上海' 将被使用。
当地时间是现在：周二年12月17  18：22：10 CST 2013 。
通用时间是现在：周二年12月17  10：22：10 UTC 2013 。
以上信息是否正常？
1 ）是
 2 ）否
＃？1

您可以通过追加该行来使此更改永久保留
        TZ = ' 亚洲/上海' ; 出口TZ
到文件“ .profile文件”  在你的home目录; 然后注销了和日志中试。

这里是 TZ值再次，这个时候在标准输出上就可以了
可以在shell脚本中使用 / usr / bin / tzselect命令：
亚洲 /上海
```

6.复制文件到/等目录下

sudo cp / usr / share / zoneinfo /亚洲/上海/ etc / localtime

7.更新时间

sudo ntpdate time.windows.com





修改时间

```
sudo date -s MM / DD / YY // 修改日期 
sudo date -s hh：mm：ss // 修改时间
```

在修改时间以后，修改硬件CMOS的时间

```
sudo hwclock --systohc // 非常重要，如果没有这一步的话，后面时间还是不准
```




# bash 脚本

## echo格式

1.echo 具有给输出的字符加颜色的功能，格式如下：

格式: echo -e "\033[字背景颜色;字体颜色;ANSI控制码m字符串\033[0m" 

-e选项是让echo能够识别转义字符，否则不能显示颜色，先上个格式相关的例子

例1：

echo -e "\033[41;36m something here \033[0m" 

或者：echo -e "\033[36;41m something here \033[0m"

输出结果一样：

![img](http://blog.csdn.net/hepeng597/article/details/7743853)![img](http://my.csdn.net/uploads/201207/13/1342160763_1993.JPG)

![img](http://blog.csdn.net/hepeng597/article/details/7743853)

其中41的位置代表底色为红色, 36的位置是代表字的颜色为深绿色（在底色为红色下看不出来）,41和36的位置交换也没有关系。

例2：

或者：echo -e "\033[41;36;1m something here \033[0m" 

或者：echo -e "\033[36;41;1m something here \033[0m"

输出结果：

![img](http://my.csdn.net/uploads/201207/13/1342160844_4440.JPG)

该格式加了“1m”参数，表示“设置高亮度”，从图中可以看出亮度不同，例1和例2格式的结尾都有“0m”，表示“关闭所有属性”，如果不设置0m，那么接下来的输出都会延续上个输出属性。

2.参数及对应的颜色

| 字颜色                                      | 字背景颜色                                    | ANSI控制码的说明                               |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| 30:黑 31:红 32:绿 33:黄 34:蓝色 35:紫色 36:深绿 37:白色 | 40:黑 41:深红 42:绿 43:黄色 44:蓝色 45:紫色 46:深绿 47:白色 | 033[0m 关闭所有属性 \033[1m 设置高亮度 \033[4m 下划线 \033[5m 闪烁 \033[7m 反显 \033[8m 消隐 \033[30m -- \33[37m 设置前景色 \033[40m -- \33[47m 设置背景色 \033[nA 光标上移n行 \033[nB 光标下移n行 \033[nC 光标右移n行 \033[nD 光标左移n行 \033[y;xH设置光标位置 \033[2J 清屏 \033[K 清除从光标到行尾的内容 \033[s 保存光标位置 \033[u 恢复光标位置 \033[?25l 隐藏光标 \033[?25h 显示光标 |

3.有了以上的知识，我们就可以写一个通用的函数了

```
[cpp] view plain copy
#!/bin/bash  
  
function _color_()  
{  
    case "$1" in  
        red)    nn="31";;  
        green)  nn="32";;  
        yellow) nn="33";;  
        blue)   nn="34";;  
        purple) nn="35";;  
        cyan)   nn="36";;  
    esac  
    ff=""  
    case "$2" in  
        bold)   ff=";1";;  
        bright) ff=";2";;  
        uscore) ff=";4";;  
        blink)  ff=";5";;  
        invert) ff=";7";;  
    esac  
    color_begin=`echo -e -n "\033[${nn}${ff}m"`  
    color_end=`echo -e -n "\033[0m"`  
    while read line; do  
        echo "${color_begin}${line}${color_end}"  
    done  
}  
#__main__  
echo "this is a test" | _color_ blue bold  

```



**获取192.168.1.1~192.168.1.254 可已连接的IP地址**

附加知识：信号列表，使用`trap`捕获

![](https://ws2.sinaimg.cn/large/006tKfTcgy1flq58elxcwj310s0ce0tr.jpg)

```shell
#!/bin/bash
#查看是否有上一次的记录，如果有就删除
if [ -e ok.txt  ]
then
rm -rf ok.txt
echo "old ok.txt has deleted!"
fi

#设置初始状态，当接受到信号后，会改变此变量
state=ok
#捕获信号
trap "state=exit" 1 2 3 24

mode=192.168.1.
begin=1
end=254

for ((i=$begin;i<=$end;i++))
do

#注意到"$test"x最后的x，这是特意安排的，因为当$test为空的时候，上面的表达式就变成了x = testx，显然是不相等的。而如果没有这个x，表达式就会报错：[: =: unary operator expected 

if [ x$state == x"exit" ]
then
echo -e "\033[41:36m program is stopped... \033[0m"
break
fi

ip=$mode$i
ping -c 1 $ip >> /dev/null
re=$?

if [  $re -ne 0 ]
then
echo $ip error
else
 echo -e "\033[7m$ip \033[0m" | tee -a ok.txt
fi

if [ $i -eq $end  ]
then
 echo  program has completed
echo -e "\033[2J program has completed,here are the ok ip list ,you can also find it from ok.txt: \033[0m"
echo -e "\033[7m `cat ok.txt`\033[0m"
fi

done
```

