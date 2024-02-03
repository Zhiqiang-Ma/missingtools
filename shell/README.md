

### 思路流程

~~~ mermaid
graph TB
文件查找find[查找文件 find] --> 参数化xargs[参数化xargs]
参数化xargs & 内容查找grep[查找内容 grep] --> 管道[管道操作]

管道 --> 取某列操作cut[取某列操作cut]
取某列操作cut --> 替换sed[替换sed] & 替换tr[替换tr] & 行操作awk[行操作awk]

替换tr & 替换sed & 行操作awk --> 排序sort[排序sort]
排序sort --> 列拼接paste[列拼接paste] & 行列拼接cat[行列拼接cat]

~~~

### 1，find命令 

* 按文件查找

查找分为 **文件查找和内容查找**，文件查找主要的命令为find命令，内容查找主要为grep命令。

~~~ shell
find 参数命令 ‘正则表达式’ 查找目录
~~~

* 按名字查找 -name

如：找到当前目录下所有不以 .txt 结尾的文件

~~~shell
find . ! -name "*.txt" -print # 查找txt文件并打印
find . ! -name "*.txt" -delete # 查找txt文件并删除
# 找到audio_dir 目录下所有以.wav 结尾的文件并重定向到文件 -i 不区分大小写
find $audio_dir -iname "*.wav" > $tmp_dir/wav.flist
~~~

* 按类型和大小 -type 和-size

~~~shell
# 查找当前目录下文件大小大于2K的文件
find . -type f -size +2k 
~~~

将查找到的结果可以通过管道命令进行向下传输，进行下一步的操作。如果是要将查找到文件的内容作为操作的对象还需要通过 xargs命令

* 与xargs 结合

一般结合方式为：find 命令的 -print0 选项生成以空格（‘\0’）作为分隔符的输出，然后将其作为xargs命令的输入。

~~~shell
# 统计dir目录下所有.c 文件的行数
find dir -type -f -iname '*.c' print0 | xargs -0 wc -l
~~~

* 与-exec 结合

为查找到的每个文件名执行命令

~~~shell
find . -name *.cpp -exec sed -i 's/ABC/abc/g' \{\} \;
~~~

### 2，grep内容查找

grep主要是查找文件内容，或是管道传入内容的行中是否符合，是输出，不是继续查找直到结束。egrep 是使用扩展正则表达式。

~~~shell
grep ‘\[\]’ 
~~~

* 仅显示查找到内容 -o

其他参数命令 -o （only）和 -v（invert）

其中 -o 参数只打印匹配到的文本

~~~shell
echo this is a line. | egrep -o "[a-z]+\."
line
~~~

* 取反操作 -v

-v 参数打印出不匹配正则的行

~~~shell
echo this is a line. | egrep -v "[a-z]+\."
this is a 
~~~

* 执行多次 -e

-e 参数 匹配多个模型

~~~shell
$ grep -e "pattern1" -e "pattern2" 文件
~~~

* 精确查找 -w

使用 `grep` 命令来匹配确切的模式。为此，我们使用带有 `-w` 选项的 `grep` **命令**

* 按组查找 -Fwf

-F 为使用 fgrep 为快速查找，适用于查找特定的字符串而不适用于规则查找。

fgrep 命令和带 -F 标志的 grep命令是一样的。**fgrep命令** 是用来搜索 file 参数指定的输入文件（缺省为标准输入）中的匹配模式的行。fgrep 命令特别搜索 Pattern 参数，它们是固定的字符串。如果在 File 参数中指定一个以上的文件 fgrep 命令将显示包含匹配行的文件

~~~shell
grep -F -w -f uttid_same wav.scp
~~~

fgrep 的具体参数含义：

~~~
-f StringFile：指定包含字符串的文件。
-w：执行单词搜索。
-i：当进行比较时忽略字母的大小写。
~~~

使用kaldi 中的filter_scp.pl 同样的效果

~~~
( echo foo 1; echo bar 2 ) | tools/filter_scp.pl <(echo foo)
~~~

> 参考：https://wangchujiang.com/linux-command/c/fgrep.html
>
> 参考：http://www.manongjc.com/detail/51-fszegpebioqqrwb.html

* 可以查找传入的参数

以下实现了文件内容的按行在目标文件中进行查找。注意其中文件是以追加的方式进行的，否则只显示最后一次查找的结果

~~~shell
tmp="tmp"
if [ ! -d $tmp ];then
         mkdir $tmp
fi
split -n l/$processors $words ./$tmp/
for file in `ls $tmp`;do
    while read line; do
        word=`echo $line | awk '{print $1}'`
        grep -w $line ${source} | head -n $num >> $tmp/${file}_result
    done < $tmp/$file
done &
wait
~~~

* 配合xargs 命令的查找

~~~shell
cat $file | xargs -I {} grep -w -B1 "row_{}" $row_file > $result 
~~~

注意要查找的内容需要加上特定的字符进行拼接 "row_{}"

* grep 匹配空行

~~~shell
 grep "^$" tmp | wc -l
 grep -n "^$" tmp
~~~

* 搜索文件中的特定字符

~~~shell
grep -r "KALDI_PARANOID_ASSERT" /path/to/kaldi/src
~~~



### 3，cut 取特定列操作

有时查找到的内容中有多个字段即多列，此时需要将一列单独拿出处理。这时候需要cut进行取特定列操作。默认是以 \t 进行分隔的。

~~~shell
# 去除文件的第二列之后的所有列
cut -d " " -f2- file
~~~

设置分隔符 -d 

### 4，sed替换删除 

sed 的动作说明

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何东东；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- s ：取代，可以直接进行取代的工作，通常s 的动作搭配正规表示法！如 's/old/new/g '就是啦！

sed 命令，流编辑器最常用的为文本替换

* 删除特定的行

~~~shell
# 最后的g表示进行全局的替换操作,sed会将s之后的符号视为默认分隔符
$ sed 's/pattern/replace_string/g' file
~~~

-i  修改后的数据替换原始文件（慎用，提前备份）

~~~shell
$ sed -i 's/pattern/replace_string/g' file
~~~

* 删除空格

~~~shell
$ sed 's/[ ]\+/ /g' file # 删除多个空格
~~~

* 删除末尾空格

~~~shell
sed ’s/[ ]*$//g‘ file 
~~~

* 删除空行

~~~
$ sed '/^$/d' file
~~~

* 按时间查找，特定时间段的日志

~~~shell
// -n 表示只显示操作后结果
sed -n '/2020-08-04 14:46:07/,/2020-08-04 14:46:12/p' all.log
//如果不知道具体的时间，只能精确到分钟可以使用模糊查找
sed -n '/2019-10-24 22:14:*/,/2019-10-24 22:16:*/p' all.log
~~~

* 配合grep找到特定时间段日志的ERROR

~~~shell
sed -n '/2019-10-24 22:16:21/,/2019-10-21 20:16:58/p' all.log | grep 'ERROR'
~~~

* 替换指定关键词

~~~shell
sed 's/\(iat000.*64\)/D:\/cer_wavs\/wavs\/\1/g' wer_80.log > add_wer_80.log
~~~

* 在一行的开头和结尾插入字符

~~~shell
// 在文件开头插入
sed "s/^/HEAD/g" test.file
// 在每行末尾插入
sed "s/$/TAIL/g" test.file
// 将两行何在一起
sed "/./{s/^/HEAD/;s/$/TAIL/}" test.file
~~~

* 删除所有标点

~~~shell
sed 's/[[:punct:]]/ /g' tmp
~~~

* 删除特定第几行

~~~shell
sed '3d,6d,10d' text # 删除不连续的行 第三，六，十行
sed '3,6d' text # 删除连续的行，第三到六行
sed '3,6!d' text # 删除连续的行，除了第三到六行,其他的行
sed '$d' text # 删除最后一行
~~~

* 打印特定的行

~~~shell
sed '3,6p' text # 删除不连续的行 第三，六行
~~~

* 删除包含特定字符的行

~~~shell
sed '/pattern/d' text
~~~

* 英文之间加上特定符号

  ~~~shell
  paste <(cut -f1 text_org) <(cut -f2- text_org | tr '[a-z]' '[A-Z]'| sed 's/\([A-Z]\) \([A-Z]\)/\1▁\2/g' | sed 's/\([A-Z]\) \([A-Z]\)/\1▁\2/g' | tr -d " " ) | head 
  ~~~

  **注意：sed 使用括号，可以进行分组操作。**

* 传入参数

  将单引号改为双引号

  ~~~shell
  sed -n "/${sessionID}-START/,/${sessionID}-END/p" t.t
  ~~~


### 5，tr 命令

注意：该命令只能通过标准输入接收输入，调用格式如下：

~~~shell
# 按照位置从第一个位置将set1 映射为set2
tr [options] set1 set2
~~~

* 替换空格

~~~shell
tr -d " "
tr " " "\n"
tr '[a-z]' '[A-Z]' text
~~~

* 合并多个空格为一个 

~~~shell
echo "a   b    c   d" | tr -s ' '
~~~

### 6，awk行操作

awk 可以处理数据流，其逐行的处理文件，BEGIN之后的语句最先执行，对于匹配pattern的行会对其执行commands命令，最后整个文件处理完成后，awk会执行END之后的命令。

~~~shell
$ awk 'BEGIN{print 'start'} pattern{commands} END{print 'end'}' file
~~~

* 特殊变量

NR（number row）相当于当前行号

NF（number file）相当于处理当前行的字段数量。默认字段分隔符为空格

$0：包含当前记录的文本内容

$1：该变量包含第一个字段的内容

-F 指定不同的分隔符

~~~shell
//输出第一列到新文本中
awk '{print $1}' file 
// 以：冒号为分隔，输出最后一列
awk -F: '{print $NF}' file
// 以/ 为分隔，输出最后一列和倒数第二列，两列之间用，逗号隔开
awk -F '/' '{i=NF-1;printf("%s %s\n",$NF,$i)}' filename
// 打印多列并在中间插上特定字符串
awk '{print $1 ":" $3}' filename
~~~

输出文本的行数：

~~~shell
# END 代表最后执行，NR代表当前的行数 (Number Row)
awk 'END{print NR}'
# 利用此特性生成字典的index
awk '{print $0 " " NR+1}'
~~~

输出某文件的特定行数，比如100行到1000行

~~~shell
awk 'NR==100, NR==1000{print}' file
~~~

打印位于模式start_pattern与end_pattern之间的文本

~~~shell
awk '/start_pattern/,/end_pattern/' filename
~~~

计算某一列的和。其中的参数-F指定分隔符

~~~shell
awk -F: 'BEGIN{sum=0}{sum+=$1}END{print sum/3600}' filename
~~~

* awk 传递参数

情景，在使用for 和awk 的时候，无法在awk的命令语句中使用$i，不能正确的使用i的值。这时候需要通过外部进行传递。比如当wer大于$i 小于$i+20 的时候：

~~~shell
for ((i=0;i<100;i+=20));do
	grep -w 'WER' $temp/$row_file | awk '{if($3>=v1 && $3<v2){print $1}}' v1=$i v2=$((i+20)) > $temp/text
done
~~~

注意以上的v2=$((i+20)) 先计算在取值

* awk 计算小数除法

~~~shell
# 以下结果为0，不支持浮点数计算
var1=3
var2=10
var3=`expr $var1/$var2`
# method 1：使用
var=`echo "scale=2;$t1/$t2" | bc`
# method 2
awk 'BEGIN{printf "%.2f%\n",('$t1'/'$t2')*100}'
~~~

* 数组统计文件中特定词个数

~~~shell
awk -F“ ” '{a[$1]++} END{for(i in a){print(a[i],i)}}' label_new | sort -nr
~~~

note: 注意这里的for中括号的位置！

直接的建立数组并且将值\$1作为key，value使用a[$1]++ ，自动加一，遍历结束后，在END 中进行数组的输出。通过管道对其进行排序。

~~~shell
awk '{for(n=2;n<=NF;n++) print $n; }' | sort | uniq -c |sort -nr
~~~

uniq -c 数数并显示重复的行数，然后在根据数字进行逆序排列

* 判断每个字符的长度

~~~shell
awk '{if(length($2)>=3){print $0 }}' text  # 判断第二类为大于一个字的，就输出
~~~

~~~shell
awk '{if($1>200 && $1<500){print $2}}' text # 与运算符
~~~

### 7，sort & uniq 排序

* 查找重复行

~~~
sort file |uniq -d
~~~

* 随机的挑选

~~~shell
sort -R filename | head -n 100
shuf -n 100 filename
~~~

* 随机的选取文件N行

~~~shell
# 使用sort 命令将文件随机排序，选择前100行
sort --random-sort file | head -n 100
~~~

还可以使用

~~~shell
shuf -n 20
~~~

* 指定特定分隔

~~~
sort -t ":" -k3 text > tmp
~~~

### 8，cat & paste 按行或列拼接

按行将文件拼接在一起

~~~shell
$ cat file1 file2
~~~

paste

按列合并多个文件

~~~shell
# -d 用来指定分隔符，默认为制表符 \t
$ paste -d ' ' file1 files
~~~

拼接两个标准输入

~~~shell
paste -d "," <(sed file1) <(sed file2) > outputfile
paste -d " " <(cut -d " " -f 1 text) <(cut -d " " -f 1 text) >utt2spk
~~~

### 9，xargs 输入作为命令参数

~~~shell
echo "hello world I am fine" | xargs -n 1 echo
// 结果
hello
world
I
am
fine
~~~

输出 wav 文件下面，音频信息

~~~shell
 ls ./wav | xargs -I {} sox ./wav/{} -n stat > duration.log 2>&1
~~~

参数-I 用于指定替换的字符串这个字符串会在xargs解析输入时被参数替换掉。将-I 和argvs 一起使用，对于每一个参数指定的命令只会执行一次。

参数 -d 用于指定分隔符。 -n 用于一次显示几个显示

* xargs 和grep 的结合

选取文件中特定关键词的上下文本

如果文本内容不存咋唯一的编号，先对其进行编号处理 比如 row_1, row_2 etc

~~~shell
awk '{print 'row_'$1,$2}' filename1 > filename2
~~~

选取带有关键字的行的编号 即 row_1, row_2 但是只保留阿拉伯数字

~~~ shell
grep -w ‘keyword_WER’ filename2 > filename3
~~~

理由awk 进行过滤， 如果每行的第二个字段的数字大于80，则输出字段1行号

~~~shell
awk '{if ($2>80) {print $1}}' filename3 > filename4
~~~

去掉行号的 row_ 剩下 阿利伯数字

~~~shell
cut -d " " -f 2 fielname4 > filename5
~~~

组合xargs 和grep 进行查找文件。xargv 的参数 -I 表示了需要使用替换，即xargs的输出替换后面{} 的位置的内容。注意的是必须要在“row_{}” 使用row定位符号来告诉 grep 在这里需要替换。

注意：此命令的特点是按照id文件逐条的进行查找，所以结果文件也是按照id文件的顺序。而grep -Fwf 是按照第一列进行排序。

~~~shell
cat filename5 | xargs -n 1 -I {} grep -w "row_{}" -A3 -B2 > final.log
~~~

按照id 过滤

~~~shell
grep -F -w -f uttid_same wav.scp
~~~

利用for循环进行查找

~~~shell
#!/bin/bash

cat -A ref.txt | while read id;
do
    real_id=`echo $id| cut -d"^" -f1`
    grep $real_id target.txt >>ref.gff
done
~~~

`cat`提取整个文件内容;用`|`管道符号传给 `while read`读取，
 由于window下的回车符号与Linux下的回车不一样：window下的回车到了linux下会多出`^M`,所以在代码中用`cut -d"^"`分割文件，并取第一部分作为ID

### 10，split 命令

将一个大文件分解为多份,

其中 -n 表示 桉行尽心分割 后面 l/10 ,是说均分成10 分的行数。tmp 后面的 "/" 不能丢，否则报错

~~~shell
split -n l/${numprocessers} ${test_file} tmp/
~~~

### 11，获取今天的日期

~~~shell
DATE=$(date +%Y%m%d)
~~~

### 12，head & tail

* 显示前n 行

~~~
head -n 5 text
~~~

* 除了最后的n行都显示

~~~
head -n -5 text
~~~

* 显示最后的n行

~~~shell
tail -n 3 text
~~~

* 显示从第3行到末尾

~~~shell
tail -n +3 text
~~~

* 动态的显示循环读取 -f

~~~shell
tail -f log
~~~

* 每个文件输出2000行

~~~shell
for i in `seq 2000 2000 100000`; do
	head -n $i text | tail 2000 > ${i}_segged 
done
~~~

### 13，A中有B中无

用于挑选两个文件相同行，或不同行。

~~~SHELL
#a，b两个文件，
a中有，b中无的行：
cat a b b | sort | uniq -u
a，b中都有的行：
cat a b | sort | uniq -d
~~~

### 14，parallel 并行任务

~~~shell
parallel -j 10 sed -i ‘s/foo/bar/g' {} ::: $(cat filelist.txt)
~~~

参数介绍

~~~shell
::: 后面接参数
:::: 后面接文件
-j、--jobs   并行任务数
-N  每次输入的参数数量
--xargs会在一行中输入尽可能多的参数
-xapply 从每一个源获取一个参数（或文件一行）
--header  把每一行输入中的第一个值做为参数名
-m   表示每个job不重复输出“背景”（context）
-X   与-m相反，会重复输出“背景文本”
-q  保护后面的命令
--trim  lr 去除参数两头的空格，只能去除空格，换行符和tab都不能去除
--keep-order/-k   强制使输出与参数保持顺序 --keep-order/-k
--tmpdir/ --results   都是保存文件，但是后者可以有结构的保存
--delay  延迟每个任务启动时间
--halt  终止任务
--pipe    该参数使得我们可以将输入（stdin）分为多块（block）
--block  参数可以指定每块的大小
~~~

* 并行查找

~~~shell
cat bigfile.txt | parallel --pipe grep 'pattern'
~~~

### 16，set 设置执行格式

* set -u 

专门针对变量的模式，如果有未赋值定义的变量，通常对程序意味着冗余，无效，这不是我们所希望的事情。（如果，shell使用了大量的变量的话），或者，某个变量为空，而在脚本内rm -rf 变量，此时，set -u将会保护你，因为，如果为空，而又没有-u，rm -rf 命令将会删除一切，这个时候，你设置了set -u 可能会救你一命！！！

* set -e （error）

**遇到错误就停止运行，退出脚本，后面的命令不再执行。**

debug断点模式，这个在脚本的流程控制中常常用到，比如，某一个脚本，不希望见到任何错误（返回值非零表示错误），因为这个错误对于脚本工作很重要，比如安装脚本，前面都错了，后面的行还在执行，有可能造成灾难性的后果，这将大大的提高脚本的健壮性。

* set -o pipefail

管道命令参与debug断点模式，shell默认会认为管道命令是一个整体，是与 | 的关系，set -o pipefail 更改为与或||，也就是管道命令参与。

* set -x

显示脚本执行过程，并显示脚本对变量的处理结果。如果，某一个脚本使用了大量的变量，而我们希望能看到这些变量的传递，使用是否正确，那么，set -x 将是你很好的选择。（快速定位问题，尤其是变量所产生的问题）

综上，set -ue 和 set -o pipefail 可以保证shell脚本的健壮性！！！set -x 可以为你提供可视化的变量值检查。如果有危险命令，比如> 重定向，rm -rf 删除，这些，请尽量使用这些set。

### 18，dmesg 查看内存溢出 

按时间查看内核信息情况。dmesg 命令主要用来显示内核信息

~~~shell
dmesg -T 
~~~

###  20，禁止产生core 文件

~~~shell
ulimit -S -c 0 > /dev/null 2>&1 
~~~

### 21，删除 win 下面的 换行 ^M

在vim 命令行模式下输入：

~~~shell
 %s/\r//g
~~~

### 22，正则表达式

~~~shell
# 字符簇含义
[[:alpha:]] 任何字母
[[:digit:]] 任何数字
[[:alnum:]] 任何字母和数字
[[:space:]] 任何白字符
[[:upper:]] 任何大写字母
[[:lower:]] 任何小写字母
[[:punct:]] 任何标点符号
[[:xdigit:]] 任何16进制的数字，相当于[0-9a-fA-F]
~~~

\s 表示字符  \d 表示表示数字 

一般结合 sed 进行替换。比如去除标点

~~~shell
sed 's/[[:punct:]]/ /g' file_name
~~~

### 23，反引号使用

反引号的作用，一般是将变量或是表达是的值在重新付给新的变量

~~~shell
var1=12
var2=3
vars=`$var1 / var2`
~~~

用于表达式运算

~~~shell
rank=`expr $model_rank \* $num_gpus+$i`
~~~

注意赋值语句之间没有空格

~~~shell
# 赋值之间没有空格
name="allan"
temp=$name
~~~

### 24，pwd 和PWD

有“/”表示绝对路径，从站点的根目录开始找
没有“/”表示相对路径，从当前路径开始找。

~~~shell
/speech_data/aidatatang
# 表示从跟目录开始查找
speech_data/aidatatang
# 表示从当前目录查找，当然找不到了！！
~~~

绝对和相对路径

~~~shell
$pwd # 相对路径
$PWD # 绝对路径
~~~



### 25，/dev/null

可以将/dev/null看作"黑洞". 它非常等价于一个只写文件. 所有写入它的内容都会永远丢失

禁止标准输出.  1 cat $filename >/dev/null  # 文件内容丢失，而不会输出到标准输出

命令 1>/dev/null 2>&1的含义

\> 代表重定向到哪里

1 表示stdout标准输出，系统默认值是1，所以">/dev/null"等同于"1>/dev/null" 

2 表示stderr标准错误

& 表示等同于的意思，2>&1，表示2的输出重定向等同于1

### 26，basename & dirname

即去除文件名的目录部分和后缀部分。返回一个字符串参数的基本文件名称。

~~~shell
basename /usr/bin/sort
# 输出 sort
basename include/stdio.h .h
# 输出 stdio
~~~

dirname命令去除文件名中的非目录部分，删除最后一个“\”后面的路径，显示父目录。脚本可以使用 dirname $0来定位脚本的路径

~~~shell
dirname $0
~~~

### 27，动态查看日志 tail -f

用于实时监视文件内容的变化。当使用tail -f filename命令时，tail会显示文件的最后几行内容，并且当文件内容发生变化时，tail会继续显示新的内容

~~~shell
tail -f log
~~~

-f 的缺点为：如果文件被移动或重命名，tail -f将无法继续跟踪文件的变化，因为它仍然绑定到原始的文件描述符上。

-F也可以用于实时监视文件内容的变化。但是，与-f不同的是，-F会尝试重新读取无法立即打开的文件。这是因为-F选项是按照文件名来跟踪文件的，而不是文件描述符。因此，即使文件被移动或重命名，只要文件名保持不变，tail -F就能够继续显示文件的新内容

### 28，统计文件夹下文件个数

~~~shell
ls –l | grep “^-”| wc –l 
~~~

查看文件内容行

~~~shell
cat text| wc -l
~~~

### 29，tar压缩我解压文件

* 基本压缩和解压

~~~shell
tar -cvf test.tar test
~~~

参数 f 后跟压缩后的文件名，习惯上用tar后缀， 最后参数为要压缩的目录。如果以gzip压缩，需要加上 -z 参数，解压时候也需要带上 -z 。

~~~shell
tar -zcvf test.tgz test
~~~

 **解压**

~~~shell
tar -xvf test.tar
tar -zxvf test.tgz
~~~

* 压缩时排除文件夹

压缩TVdata，但是不想压缩TVdata下面的 .backup 文件夹，此时可以使用

~~~shell
tar -zcvf TVdata.tgz --exclude=TVdata/.backup TVdata
~~~

注意 --exclude 参数必须在中间。

* 解压时候删除特定目录

解压删除前导组件,压缩文件eg.tar 中文件目录信息为  “src/src/src/eg.txt”

~~~shell
tar -xvf eg.tar --strip-components 1
~~~

结果：src/src/eg.txt

~~~shell
tar -xvf eg.tar --strip-components 3
~~~

解压结果为： eg.txt

* 解压到指定的目录

如果想指定解压目录，可以加参数-C 目标目录

~~~shell
tar -xvf eg.tar -C /data/dst
~~~

* 解压参数解释

~~~shell
 -c: 建立压缩档案
 -v：输出信息
 -x：解压
 -t：查看内容
 -r：向压缩归档文件末尾追加文件
 -u：更新原压缩包中的文件
~~~

### 30，zip 压缩

它最大的优点就是在不同的操作系统平台上使用。缺点就是支持的压缩率不是很高，而tar.gz和tar.bz2在压缩率方面做得非常好。

~~~shell
zip -r archive_name.zip filename #-r是压缩文件
~~~

解压文件在当前文件下

~~~shell
unzip archive_name.zip
~~~

解压文件可以将文件解压缩至一个你指定的的目录，使用-d参数

~~~shell
unzip archive_name.zip -d new_dir 
~~~

### 31，ln 建立软连接

使用格式为：

ln –s 源文件目录 连接文件目录

~~~shell
ln –s wenet/lm lm 
~~~

* 修改软连接

~~~shell
ln –snf 源文件目录 连接文件目录
~~~

* 删除软连接

~~~shell
rm –rf ./wenet 
~~~

注意 rm –rf ./wenet/ 多加一个斜线会将源数据删除，谨慎使用

### 32，env 查看端口号是否开启

~~~shell
env | grep 11013 # 查看配置11013 的端口
~~~

查看配置代理

~~~C++
env | grep -i proxy
~~~

### 33，lsof 查看端口占用

系统中的端口20021 是否被进程占用

~~~shell
lsof -i:20021
~~~

* 批量kill 特定进程

~~~shell
kill `ps -aux | grep "wenet/bin/train.py" | awk '{print $2}'`
~~~

* 查看网络

~~~shell
netstat -rn
~~~

* 查看端口状态

~~~shell
netstat -a | grep port
~~~

容器的默认分配地址为：172.17.0.x/24

TODO 不同的容器见是怎样进行通信的

* 查看端口是否启用开放

~~~
telnet 10.18.218.219 1988
~~~

即给定的主机上的端口1988 是都启用



### 34，ssh免密登录

step1: 在本地生产秘钥，在 ./ssh 中查看

~~~shell
ssh-keygen
~~~

step2: 在需要免密登录的服务器上生产秘钥，在 ./ssh 中查看

step3：拷贝本地公钥在服务器上

~~~shell
ssh-copy-id name@10.12.218.201
~~~

### 35，cp 复制命令

* 目标文件夹不存在

~~~shell
cp -r dir1 dir2
~~~

* 目标文件夹存在

~~~shell
cp -r dir1/. dir2
~~~

* 同名的不会覆盖

~~~shell
cp -r -n /source_adr /target_adr
~~~

### 36，mv 移动文件和改名

* 改名

如果源文件和目标文件在同一目录中，那就是改名

~~~~shell
mv bols lmls #把 bols 改名为 lmls
~~~~

* 不同的目录 进行移动

~~~shell
mv movie/ /tmp
~~~

### 37，file 查看文件编码

* 查看编码

~~~shell
file filename.txt
~~~

输出为：output: filename.txt UTF-8 Unicode text, with escape sequences

* 更改编码格式

比如将一个UTF-8 编码的文件转换成GBK编码

~~~shell
iconv -f UTF-8 -t GBK file1 -o file2
~~~

### 38，查看磁盘使用情况

* 剩余空间

~~~shell
df -h
~~~

* 文件夹已经使用的大小

~~~shell
du -h --max-depth=1
~~~

* 查看文件夹使用空间

~~~shell
du -sh ./*
~~~

注意不同的系统可能命令的方式不同

### 39，ps 查看进程运行状态

ps 命令就是最基本同时也是非常强大的进程查看命令。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多 的资源等等

~~~shell
ps -aux | grep "name" 
~~~

* 参数解释

-a ： 显示现行终端机下的所有程序，包括其他用户的程序

-f  ： 用ASCII字符显示树状结构，表达程序间的相互关系

-e   :   此参数的效果和指定"A"参数相同

### 40，nohup 后台运行命令

nohup 是 no hang up 的缩写。不挂断的运行，并没有后台运行的功能。就是指用nohup运行命令可以使命令永久的执行下去，和用户终端没有关系，并没有后台运行的意思。

* 使用示例

~~~shell
nohup bash /usr/bin/home/test.sh > /home/test.log 2>&1 &
~~~

注意后面的 & 不可省略，表示后天运行。

* 2>&1 的解释

2>&1是将标准错误（2）重定向到标准输出（&1），标准输出（&1）再被重定向输入到log 文件

### 41，diff 比较两个文件夹不同

~~~shell
diff -qr lib_o/ libs/
~~~

也可以使用 ， 但是没有尝试过

~~~shell
rsync -rv --size-only --dry-run /my/source/ /my/dest/ > diff.out
~~~

### 42，rsync 同步文件

~~~shell
rsync -avz --exclude '.*' TVDATA speech@10.18.219.161:/cephfs/speech_data
~~~

* 参数解释

-a :	这是归档模式，表示以递归方式传输文件，并保持所有属性，它等同于-r、-l、-p、-t、-g、-o、-D

-v :	表示打印一些信息，比如文件列表、文件数量等。

-z:	加上该选项，将会在传输过程中压缩。

-i:	显示改变信息

--exclude :	排除文件

### 43，wget 下载文件

使用：

~~~shell
wget options url
~~~

* 参数

-o：	重命名下载文件

-c：	断点续传

-i：	 多个文件下载

-b :	 后台下载	

-P：	指定下载目录

~~~shell
wget -P dirs url
~~~

### 44，显示特定行

从第5行开始，显示 3行的数据

~~~shell
cat tmp | tail -n +5 | head -n 3
~~~

从第5行开始，显示到底10行

~~~shell
cat tmp | head -n 10 | tail -n +5
~~~

tail -n 后面的数字加上“+” 表示从第5行开始显示，不加的时候表示从后向前数显示5行。

### 45，{ } 中括号

* 表示并的关系，例如

~~~shell
rm {lm,util}/*.o 2>/dev/null
~~~

删除 lm 和util 文件中所有的 .o 文件

* 变量替换

~~~
CXX=${CXX:-g++}
~~~

Shell 变量替换的语法，表示如果变量 C++ 没有被设置或者为空，则使用默认值 g++ 进行替换。具体的含义如下：

如果变量 `C++` 已经被设置，并且不为空，则使用变量 `C++` 的值；

如果变量 `C++` 没有被设置或者为空，则使用默认值 `g++` 进行替换

之后 ${CXX} 就表示了 g++。注意冒号和减号都是必须的，且中间没有空格。

如果没有找到匹配的字符串 `test.cc`，则返回原始字符串 `i`

应用场景，编译除了以test.cc 和main.cc结尾的文件

~~~shell
for i in util/double-conversion/*.cc util/*.cc lm/*.cc; do
	if [ "${i%test.cc}" == "$i" ] && [ "${i%main.cc}" == "$i" ]; then
		do something
	fi
done
~~~

* ${name%test.cc} 变量替换语法

\`${name%test.cc} 是一种 Shell 变量替换的语法。表示从变量 name 的末尾开始，删除匹配模式 test.cc的最短部分，具体如下

从变量 `i` 的末尾开始，查找字符串 `test.cc`

如果找到了匹配的字符串 `test.cc`，则删除匹配的最短部分，返回剩余的字符串

* 获取变量的长度

\`${#变量名}` 是一种获取变量长度的方式

~~~
NPLM="hello"
echo ${#NPLM}
~~~

以上的结果输出为 5。

### 46，( ) 小括号 

表示 指令群组（command group）

~~~shell
(cd ~ ; vcgh=`pwd` ;echo $vcgh)
~~~

指令群组有一个特性，shell会以产生subshell来执行这组指令。因此，在其中所定义的变数，仅作用于指令群组本身

### 47，连接字符

~~~shell
object=""
for i in `ls $test`; do
	object="object $i"
done
~~~

 test 为一个目录。实现的功能为，将test 文件夹下面的文件使用 “ ” 空格进行拼接。

### 48，ldd 查看库依赖

ldd 是 list, dynamic, dependencies 的缩写。

在制作自己的发行版时经常需要判断某条命令需要哪些共享库文件的支持，以确保指定的命令在独立的系统内可以可靠的运行

~~~shell
ldd -r websocket_main
输出：
linux-vdso.so.1 (0x00007ffeeffc3000)
libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f8e631c7000)
libacl.so.1 => /lib/x86_64-linux-gnu/libacl.so.1 (0x00007f8e62fbe000)
~~~

 在 ldd 命令打印的结果中，“=>”左边的表示该程序需要连接的共享库之 so 名称，右边表示由 Linux 的共享库系统找到的对应的共享库在文件系统中的具体位置。

默认情况下， /etc/ld.so.conf 文件中包含有默认的共享库搜索路径。

注意：在修改了 /etc/ld.so.conf文件或者在系统中安装了新的函数库之后，需要运行一个命令：ldconfig ，该命令用来刷新系统的共享库缓存，即 /etc/ld.so.cache 文件。为了减少共享库系统的库搜索时间，共享库系统维护了一个共享库 so 名称的缓存文件/etc/ld.so.cache。因此，在安装新的共享库之后，一定要运行 ldconfig 刷新该缓存。

### 49，test 判断命令输出结果

~~~bash
if test -z "$(ls | grep test1.txt)"; then
	echo "The result is empty."
else
	echo "The result is not empty."
fi
~~~

### 50，ipdb 调试python

使用命令

~~~shell
python3 -m pdb debug.py
~~~

调试参数：

~~~shell
n （next） 单步执行下一条语句
j（jump）+ 行号， 跳转执行改行命令
r  (return): 从当前函数返回
s （step）进入函数体
l（list） 列出改行上下文
b（broken）line number 打断点
c  (continue): 一直执行到断点
r（reset）重新启动调试器
q（quit）退出调试，清除调试信息
~~~

### 51，linux 中折叠路径

打开 ./bashrc 文件，找到如下命令，讲小写的 w 改为大写的 W。 保存后再source 文件

~~~shell
PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
~~~

### 52，nm 查看文件符号

nm 是 GNU Binutils 二进制工具集中的一个命令，用于显示目标文件中的符号。

这些符号可以代表变量、函数或其他类型的标识符。nm 命令的输出对于理解程序的链接行为、查找未定义的符号或调试问题非常有用。

总的来说，nm 命令是一个强大的工具，可以帮助程序员理解目标文件中的符号和链接行为。

### 53，查看系统版本

~~~shell
lsb_release -a
~~~

### 54，apt-get 相关

安装包，没有权限

~~~shell
chmod -R 777 /tmp && sed -i "s/deb /# deb /g" /etc/apt/sources.list.d/cuda.list && apt update && apt install -y vim tmux
~~~

删除软件及其配置文件

apt-get --purge remove <package>

### 55，调试try

再调试 try 的时候建议先把try 去掉，否则错误信息不会完整的暴露出来！！

### 56，md5sum 查看唯一码

~~~shell
md5sum file
~~~

### 57，watch动态的查看信息

动态查看显卡状态

~~~shell
watch -n 1 nvidia-smi
~~~

### 58，sox 查看音频文件信息

sox 关于音频文件册操作

~~~~shell
soxi a.wav
sox a.wav -n stat
~~~~

## 二，常用任务命令

### 1，加空格

目标：给星号之间加上空格，结尾不加。

思路：想把星号前后都加上空格，在去除双空格，在去除收尾空格。

~~~shell
***时区的百科 
* * * 时区的百科 
sed -e 's/\*/ \* /' -e 's/  / /g' -e 's/^ //g' text 
~~~

只给星号之间加上空格，星号和汉字自检不加空格：思路两个星号之间加上空格，在在两个星号之间加上空格。

~~~shell
***时区的百科 
* * *时区的百科 
sed -e 's/\*\*/\* \*/g' -e 's/\*\*/\* \*/g' text
~~~

### 2，定义function 函数

定义格式：

~~~shell
function function_name()
{
   statement1
   statement2
   ....
   statementn
}
~~~

实例如下：

~~~shell
#! /bin/bash

function sum()
{
  returnValue=$(( $1 + $2 ))
  return $Value
}
~~~

使用方式：

~~~shell
sum 16 4
echo $?
~~~

### 3，shell 中循环读入文件

~~~shell
while read line
do
       …
done < file
~~~

其实这是shell中while read line的一种用法：read通过输入重定向，把file的第一行所有的内容赋值给变量line，循环体内的命令一般包含对变量line的处理；

然后循环处理file的第二行、第三行。。。一直到file的最后一行。read命令也有退出状态，当它从文件file中读到内容时，退出状态为0，循环继续进行；当read从文件中读完最后一行后，下次便没有内容可读了，此时read的退出状态为非0，所以循环才会退出

while read line 是一次性将文件信息读入并按行赋值给变量line ，while中使用重定向机制,文件中的所有信息都被读入并重定向给了整个while 语句中的line 变量。

for是每次读取文件中一个以空格为分割符的字符串。

### 4，shell 中自定义日志

定义日志函数

~~~shell
function log_meg()
{
    time=`date +"%F %T"`
    echo "$time $1"
}
~~~

调用：

~~~shell
log_meg "Read the ${i} lines of the configuration file & Filter data randomly. begin!"
~~~

###  5，if 语句

* -z 判断值是否为空（zero）

在判断字符串是否为空的时候，如果字符串中间有空格，会报错。解决方案为

~~~shell
if [[ -z ${string}]]; then
	command
fi
# 判断两个条件
if [[! -z $string1 && ! -z $string2]]; then
do 
	command
fi
~~~

* -f 是否为文件 （file）

* -d 是否为目录（dir） 

~~~shell
if [ -z $file ]
then
	do something
elif
	do something
else
	do something
fi
~~~

* 判断字符是否相同

**保险：加上“x”字符**

~~~shell
if [ x$string = "xstring"]; then
	echo true
else
	echo false
fi
~~~

直接的使用等号，并且加入x是为了防止$string 为空的时候，比较也能正常的进行。

### 6，& 多线程执行任务

* 单纯多线程执行同一个命令

~~~shell
$CUDA_VISIBLE_DEVICES="0,1,2,3"
idx=0
num_gpus=$( echo  | awk -F "," '{print NF}')
for ((i=0;i<$num_gpus;i++));do
{
	gpu_id=$(echo "$CUDA_VISIBLE_DEVICES" | cut -d ',' -f$[$i+1])
}&
done
((idx+=1))
wait
# 可以通过使用 $((index % 3)) 外部定义的变量控制
~~~

& 表示后台进行，wait 表示等待所有的后台程序运行完成后，再继续的运行之后的代码 

* 利用linux 中的split 完成多线程执行

目标：将一个大文件，分割成多份执行命令

~~~shell
test_file=$1
numprocessers=$2
mkdir -p ./tmp
split -n l/${numprocessers} ${test_file} tmp/
for file in `ls ./tmp`;do
{
	./build/decoder_main 
		--input tmp/${file}
		--output tmp/${file}_result
}&
done
wait	# 等待所有的线程完成

cat tmp/*_result > $result
rm -r tmp
~~~

关键点是： split 中 -n  之后的 l/${numprocessers} 可以自动的将文件的行数除以线程数，分配到每个文件中。

需要了解其中的 l 的信息。

### 7，seq 生成序列

基本命令行形式

* 默认从1开始步长为1

~~~shell
seq 3
# 输出
1
2
3
~~~

* 指定开始和步长

~~~shell
seq 3 3 18
# 输出
3
6
9
12
15
18
~~~

结合for循环使用

~~~shell
for x in `seq 1 10`; do
	cat $x/text 
done > result
~~~

seq 中间表示步长：seq 0 2 10 即步长为2，终止到10 包括10。

找不同的文件夹内的同名文件进行合并

### 8，查找文件相同和不同行

comm 命令会一列列地比较两个已排序文件的差异，并将其结果显示出来，如果没有指定任何参数，则会把结果分成 3 行显示：第1行仅是在第1个文件中出现过的列， 第2行是仅在第2个文件中出现过的列，第3行则是在第1与第2个文件里都出现过的列

注意是排序后的文件。常用命令：

~~~~shell
comm -12 <(sort a.txt|uniq ) <(sort b.txt|uniq ) 
~~~~

参数解释：

-1：	不显示只在第1个文件里出现过的列

-2：	不显示只在第2个文件里出现过的列

-3：	不显示只在第1和第2个文件里出现过的列

补充：

~~~
diff 和 patch
~~~

查找文件的追加内容

利用diff 进行比较，选出以 “>” 开头的行，即为新增加的内容。

~~~shell
diff version1 version3 | grep ">" | cut -d " " -f 2-
~~~

### 9，shell 运行路径问题

在当前路径下运行其他目录中的脚本 cal_wer.sh

~~~text
|ASR_test
----|test1
----|test2
~~~

即在当前的目录下产生test1 和test2，而不用在脚本中显示的指明PWD，默认就是在当前执行脚本的目录，为workspace。

执行脚本的命令最好写绝对路径。

~~~shell
test_label=“test1 test2”
exc_path=/asr_pro/wenet/examples/hisense/s0/
for i in $test_label; do
	 mkdir $i
	 python3 $exc_path/tools/compute-wer.py --char=1 --v=1 /speech_data/ASR_TEST/$i/text $text/text >$i/wer
done 
~~~

### 11，动态库路径和环境变量

* 导入动态路径

~~~shell
echo $LD_LIBRARY_PATH # 查看路径
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib"
~~~

* 环境变量当前会话临时有效

运行编译好的文件需要给到其所在的路径位置，如果文件在 “/bin” “/sbin” “/usr/bin” “/usr/sbin” “/usr/local/bin” 中，因为路径已经在系统环境变量中了，终端命令行输入该软件可执行文件的文件名和参数(如果需要参数)，回车即可。使用 ，以下的命令查看当前环境变量中已经存在的环境变量地址。

~~~shell
echo $PATH 
~~~

不在需要添加路径到环境变量中去：

~~~shell
PATH=$PATH:路径1:路径2:...:路径n
export PATH
~~~

以上的命令表示可执行文件的路径包括原先设定的路径($PATH)，也包括从“路径1”到“路径n”的所有路径。当用户输入一个一串字符并按回车后，shell会依次在这些路径里找对应的可执行文件并交给系统核心执行。

* 环境变量长期有效写入文件

也写入文件：“/etc/profile”此文件是对所有的用户有效，用户主目录下的“.bash_profile”仅对当前的用户有效。

~~~shell
export PATH="$PATH:/opt/au1200_rm/build_tools/bin"
~~~

最后source 使得文件立即生效

使用命令，查看路径是否已经添加到环境变量中

~~~shell
echo $PATH
~~~

自定义环境变量

~~~shell
export IRSTLM=/asr_pro/kaldi/tools/irstlm
~~~

这样可以在shell 脚本中可以直接的使用自定义的环境变量了。

可借鉴：在写shell脚本的时候可以直接的将路径添加到环境变量PATH中或是自定义环境变量。这样调用脚本时可以直接的写脚本的名字而不用添加路径了。

### 13，cat 和EOF 键入内容

建立文件并键盘输入内容，以EOF 结束。

~~~shell
cat >file1 <<EOF
~~~

标准输出中并键盘输入内容，以EOF 结束。

~~~shell
cat >&2 <<EOF
~~~

在shell 文件中可以使用

~~~shell
#！ /bin/bash
if [ $# -eq 2]; then
	cat >&2 <<EOF
	usage:
	EOF
fi
~~~

check_exit 的写法：

~~~shell
function check_exist(){
  if [ ! -f $1 ]
  then
    echo $2 && exit 1;
  fi
  }
~~~

使用方式：

~~~shell
check_exist global_cmvn ERROR_MES
~~~

### 14，shell 传入参数模板

* shell 脚本传入参数

~~~shell
# check argv list
if [ $# -ne 2 ];then
        echo "usage: $0 <source_text> <result>"
        exit
fi

# $# 表示需要输入参数的个数，需要两个就是2，三个就是3
~~~

* 检查文件是否存在

~~~shell
file=$1
# check file exit
if [ ! -f $file ];then
        echo "please check $file do not exit!"
        exit
fi
~~~

* 检查二进制命令是否存在

~~~shell
if ! which ngram-count > /dev/null; then
 	echo "this command does not exit!"
 	exit 1
 done
~~~

### 15，rename 重命名

* 基本方法

~~~~shell
rename "s/oldname/newname/" file
rename -n "s/.zip/.gz/"  *  # 把.zip 后缀的改成 .gz后缀 
rename "s/.txt/.xls/"  *  # 将 txt 改为 xls
~~~~

-v 将重命名的内容都打印到标准输出，v 可以看成 verbose

-n测试会重命名的内容，将结果都打印，但是并不真正执行重命名的过程

* 批量文件重新命名

~~~shell
for file in `ls */*pcm `; do echo $file; tag=`echo $file | awk -F/ '{print $1}'`; tag2=`echo $file| awk -F/ '{print $2}'`;echo $file ${tag}/${tag}_${tag2} ;done
~~~

### 16，单引号和双引号

* 单引号

单引号定义字符串所见即所得，即将单引号内的内容原样输出，或者描述为单引号里面看到的是什么就会输出什么。单引号是全引用，被单引号括起的内容不管是常量还是变量都不会发生替换。

* 双引号

双引号引用的内容，所见非所得。如果内容中有命令、变量等，会先把变量、命令解析出结果，然后在输出最终内容。双引号是部分引用，被双引号括起的内容常量还是常量，变量则会发生替换，替换成变量内容。

* 无引号

不使用引号定义字符串时，字符串不能包含空白字符（如Space或Tab），需要该加引号，一般连续的字符串，数字，路径等可以不加引号。如果内容中有命令、变量等，会先把变量、命令解析出结果，然后在输出最终内容。

### 17，batch 后台运行

一共0到20 个任务，每次并行执行两个任务。并给出相应执行序号。

~~~shell
for task in `seq 0 2 20`; do
	for job in `seq 0 1`; do
	{
		num_PT=$[$task + $job]
        echo "[$num_PT pt] is testing"
        ./run_0317.sh $num_PT $job
	} &
	done
	wait
done
~~~

### 18，gcc 版本升级

* step1 : 配置源

~~~shell
vim  /ect/apt/sources.list
~~~

添加如下地址：

~~~~shell
deb http://mirrors.aliyun.com/ubuntu/ xenial main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main
~~~~

保存并更新

~~~shell
sudo apt-get update
~~~

* step2: 查看gcc 版本

~~~~shell
apt-cache policy gcc-8
~~~~

* step3：输入命令进行安装

~~~shell
apt-get install gcc-8=8.4.0-3ubuntu2
~~~

等号后面是输出Candidate的版本号，可以使用nvcc -V 查看现在被版本，此时在/user/local/bin 下面会存在两个版本。

* step4：更改默认版本

~~~shell
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 40
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 50
sudo update-alternatives --config gcc
~~~

链接 名称 路径 优先级（越低越优先）

update-alternatives 命令用于处理 Linux 系统中软件版本的切换，使其多版本共存

### 19，cmake 版本安装

* step1：下载安装文件

~~~shell
wget https://cmake.org/files/v3.15/cmake-3.15.3-Linux-x86_64.tar.gz
~~~

* step2：解压

~~~shell
tar zxvf cmake-3.15.3-Linux-x86_64.tar.gz
~~~

* step3：创建软连接

~~~shell
mv cmake-3.15.3-Linux-x86_64 /opt/cmake-3.15.3
ln -f /opt/cmake-3.15.3/bin/*  /usr/bin/
~~~

注: 文件路径是可以指定的, 一般选择在/opt 或 /usr 路径下, 这里选择/opt

* step4：查看版本

~~~shell
cmake --version
~~~

卸载旧版本 非必要 

~~~shell
apt-get autoremove cmake1
~~~

### 20，typra 中设置内联公式

文件→偏好设置→Markdown，勾选内联公式，重启typora

输入\$按Esc键会自动在后面加上一个\$，然后在这两个\$之间输入公式。



### 21，linux 终端无法输入

* ctrl+c 结束正在运行的程序 ping、telnet等

* ctrl+d 结束输入或退出shell

* ctrl+s 暂停屏幕输出

* ctrl+q 恢复屏幕输出

* ctrl+l 清屏，等同于Clear

### 22，容器内安装ssh

**Step1：映射端口**

远程服务器：启动容器，注意需要把容器的 22 端口映射出来，例如映射到 host 的 5222 端口：-p 5222:22。

**Step2：设置密码**

设置 root 账户密码：

passwd root

**Step3：安装ssh**

很多镜像都不会默认安装 ssh，所以需要在容器内安装 ssh 服务：

~~~~shell
apt update && apt install -y --no-install-recommends openssh-server
~~~~

**step4：允许root登录**

一般进入容器时使用的都是 root 账号，但是 ssh 默认是禁止 root 账号使用密码远程登录的，所以需要修改 ssh 配置文件使其允许：

~~~
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
~~~

**Step5：**

启动 ssh 服务：

service ssh start

**step6：本地测试连接**

本地：使用 VSCODE 连接，添加新的 ssh host 的时候地址这么写：

ssh root@your-server-ip -A -p 5222

这个 5222 就是第 1 步启动容器的时候设置的映射端口。之后输入密码即可，和正常的连接远程服务器一样。

### 23，跳板机传输文件

情景：主机 A要向主机C传输文件，但是不能直连，需要通过跳板机B，此时的B能够直连主机C

~~~shell
scp -o "ProxyJump username@10.18.218.xx:20024" avg_10_6.pt username@8.142.74.XX:/data/speech_data
scp -r -o "ProxyJump username@10.18.218.xx:20024" F username@47.93.123.XX:/data/speech_data
scp -P 22 –o “[跳板机名称] [用户名@跳板机IP]” 传输的文件 [目标机名称@目标机IP]:目标机目录
~~~

跳板机免密登录：

~~~shell
 ssh-copy-id -o "ProxyJump username@10.18.218.xx:20024"  username@47.93.123.xx
~~~

其中的-o 表示"PasswordAuthentication no" 表示不使用密码登录

another way：这条命令也可以，实质是一样的

~~~shell
$ scp –P 22 -J username@10.18.218.xx:20024 abc.txt [username@8.142.74.xx:/data/username/]
~~~

其中的-J 指定跳板机，-P指定端口（是对目标机进行传递的端口）u3 是跳板机，u2是目标机
