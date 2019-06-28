# Bioinformatics Data Skills
## Inspecting and Manipulating Text Data with Unix Tools
+ 纯文本数据格式分为三种：分别是以制表符分隔，逗号分隔，空格分隔。
+ 制表符分隔：生物信息学中最常用的。制表符的转义码(escape code)是\t。
+ 逗号分隔：CSV(comma-seperated values)，生物信息学中很少用。
+ 空格分隔：由于数据中间包含空格是很常见的，所以以空格分隔格式的文件比前两种使用的更少。
+ 上面这些文件格式分隔的格式只是以列分隔的，而如何分隔行呢？在linux和OS X系统中使用\n分隔行。
  而在DOS-style使用\r\n分隔行，比如说CSV文件。
### Inspecting Data with Head and Tail
+ 可以使用head -n 3 txtname或者tail来查看文件的前几行或者末尾几行。也可以使用(head -n 2; tail -n 2) <
  txtname来同时查看前几行和末尾几行。
+ 也可使用使用grep来挑选出文件中的行，然后使用管道，用head打印出符合正则表达式的前几行。
### Less
+ less可以用来检查文件以及管道程序中间结果。less txtname即可。也可以按下/，然后输入想要搜索的词。就能
  高亮的显式搜索结果。
+ 使用以下语句可以检查中间结果。
  ```linux command
  step1 inputfile | less
  step1 inputfile | step2 | less
  step1 inputfile | step2 | step3 | less
  ```
### Plain-Text Data Summary Information with wc,ls,and awk
+ wc输出一个文件的行数，单词数，char数。
  ```
  wc textname
  wc -l textname//列出行数
  ```
+ 我们可使用以下语句求出文件中所有非空的行数
  ```
  grep -c "[^\\n\\t]" text.txt
  ```
+ awk可以用来计算文件中的列数，默认将tabs和spaces作为列的分隔符。但是处理BED和GTF格式的文件只需要将tabs作为
  分隔符就可以了，所以使用以下语句
  ```
  awk -F "\t" '{print NF; exit}' Mus_musculus.GRCm38.75_chr1.bed
  ```
+ awk默认处理文件第一行，所以对于开始几行有注释的文件就需要一些技巧了。我们可以使用tail来消去前面几行。
  ```
  tail -n +6 Mus_musculus.GRCm38.75_chr1.gtf | awk -F "\t" '{print NF; exit}'
  ```
  但是我们最好可以使用grep来筛选我们需要的行，这样可以适用于更多文件。
### Working with Column Data with cut and Columns
+ 使用cut可以选出文件中特定的几列。如下
  ```
  grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f1,4,5 > test.txt
  //首先使用正则表达式筛选行数，然后使用cut选出1，4，5列输出到test.txt文件中。
  ```
  对于不是使用tabs作为分隔符的文件，而是使用delim的文件可以指定特定的分隔符
  ```
  cut -d, -f2,3 Mus_musculus.GRCm38.75_chr1_bed.csv | head -n 3
  ```
### Formatting Tabular Data with Column
+ 有一些数据不适合肉眼查看，我们可以用column -t来将数据展现为一个表格样式。这样就易于程序员阅读。
  ```
  grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f 1-8 | column -t | head -n 3
  ```
### The All-Powerful grep
+ grep非常的快，因为它的功能非常的单一。
  ```
  grep Olfr Mus_musculus.GRCm38.75_chr1_genes.txt | grep -v Olfr1413
  //从文件中挑选包含特定串的基因但是不包含Olfr1413的基因
  grep -v bioinfo example.txt
  //除去example中包含bioinfo的行
  grep -v -w bioinfo example.txt
  //除去example中包含bioinfo这个单词的行
  ```
+ grep可以统计文本中多少行符合特定的pattern
  ```
  $grep -c "Olfr" Mus_musculus.GRCm38.75_chr1_genes.txt
  27

  ```
+ grep之所以速度很快是因为它在寻找匹配行的时候，只要已找到pattern，那么这一行剩下的就不会在查找了，
  而是直接将这一行送入到standard output中。使用以下语句可以只显式满足pattern的单词。
  ```
  $grep -o "Olfr.*" Mus_musculus.GRCm38.75_chr1_genes.txt | head -n 3
  Olfr1416
  Olfr1415
  Olfr1414
  ```
+ 使用以下语句可以提取出gtf文件中的gene_id field。
  ```
  $ grep -E -o 'gene_id "\w+"' Mus_musculus.GRCm38.75_chr1.gtf | head -n 5
  gene_id "ENSMUSG00000090025"
  gene_id "ENSMUSG00000090025"
  gene_id "ENSMUSG00000090025"
  gene_id "ENSMUSG00000064842"
  gene_id "ENSMUSG00000064842"
  ```
  然后使用以下语句可以将上面得出的结果处理。
  ```
  $ grep -E -o 'gene_id "(\w+)"' Mus_musculus.GRCm38.75_chr1.gtf | \
    cut -f2 -d" " | \
    sed 's/"//g' | \
    sort | \
    uniq > mm_gene_id.txt
  ```
  这一系列语句，cut -f2 -d" "将grep得出的结果只取第二列。即只将基因id选出来。
  sed 's/"//g'有这样的用法 sed's/要被替代的字符串/新的字串/g'。所以这里将"替代为空字符。
  然后sort将结果排序，uniq去除重复行，最后输出到mm_gene_id.txt文件中。
### Decoding Plain-Text Data:hexdump
+ 存文本格式的文件是使用ascii编码的。
  ```
  $ file Mus_musculus.GRCm38.75_chr1.bed Mus_musculus.GRCm38.75_chr1.gtf
  Mus_musculus.GRCm38.75_chr1.bed: ASCII text
  Mus_musculus.GRCm38.75_chr1.gtf: ASCII text, with very long lines
  这条语句可以看到文件的编码方式。
  ```
  UTF-8是ascii的超集。
+ 使用以下语句可以看出文本文档中不属于ASCII字符的字符。
  ```
  $ hexdump -c improper.fa
  ```
  对于由人类手工生成的数据，我们要注意检查它的编码方式，可以使用hexdump,file,grep去检查。
### Sorting Plain-Text Data with Sort
+ 我们常常需要将存文本数据排序，一般有以下的理由。
  1. 有时候排序后的数据对于我们后续的文件操作可以更加的节省时间，因为操作排序好的数据是很好的。
  2. 有时候我们需要对数据去重，而排序则是去重的前驱操作。
+ cut和sort都是设计为处理纯文本数据的列。sort直接施加于文件，将文件按字母序排序。
+ 通常sort将空格视为field分隔符，如果处理其他分隔符的文件的话，比如说csv文件，那么可以使用
  -t"," option。
+ 我们也可以将文件按不同的key值排序，并且可以按照数字序排序。
  ```
  原始数据example.bed
  chr1 26 39
  chr1 32 47
  chr3 11 28
  chr1 40 49
  chr3 16 27
  chr1 9 28
  chr2 35 54
  chr1 10 19
  $ sort -k 1,1 -k 2,2n example.bed > example_sorted.bed
  chr1 9 28
  chr1 10 19
  chr1 26 39
  chr1 32 47
  chr2 35 54
  chr3 11 28
  chr3 16 27
  ```
+ 加上-r option就可以逆向排序。或者如果只想要单独的一个列逆向排序可以使用以下语句。
  ```
  $ sort -k1,1 -k2,2nr example.bed
  ```
  可以使用以下语句来检查文件是否已经按照语句规则被排好序。
  ```
  $ sort -k1,1 -k2,2n -c example.bed
  ```
+ 还有一种情况很奇怪，如下：
  ```
  原始数据example2.bed
  chr2  15 19
  chr22 32 46
  chr10 31 47
  chr1  34 49
  chr11 6  16
  chr2  17 22
  chr2  27 46
  chr10 30 42
  我们为了对文件第一列和第二列排序，可以使用以下语句
  $ sort -k1,1 -k2,2n example2.bed
  chr1  34 49
  chr10 30 42
  chr10 31 47
  chr11 6  16
  chr2  15 19
  chr2  17 22
  chr2  27 46
  chr22 32 46
  可以看出结果并不理想，因为chr sort将它看为字符串，但是以我们的角度看，明显10应该排在2后面，
  这个时候可以使用
  $ sort -k1,1V -k2,2n example2.bed
  在第一列后加上V option就可以排序字符串中的数字。
  ```
+ 有时候为了排序很大的文件，需要大量的磁盘操作，这样就会导致排序算法很慢。可以使用buffer来进行
  merge sort来进行这种大文件排序。加上一个-S2G option即可。
  ```
  $ sort -k1,1 -k4,4n -S2G Mus_musculus.GRCm38.75_chr1_random.gtf
  ```
  或者也可以使用并行算法来对这种排序。
  ```
  $ sort -k1,1 -k4,4n --parallel 4 Mus_musculus.GRCm38.75_chr1_random.gtf
  ```
### Finding Unique Values in Uniq
+ uniq只会删除连续的重复的field。如果想要将文件中所有重复的field都删除，那么就需要将文件先排序。
 ```
 $ sort letters.txt | uniq
 $ sort letters.txt | uniq -c
 可以列出每个field出现的次数。
 $ grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f3 | sort | uniq -c | \
 $ sort -rn
 上面这个语句将文件中grep筛选出来的结果先去除第三列，然后排序，然后去重并且列出个数，最后
 将结果按照逆序排序输出。
 ```
+ uniq加上-d option可以输出所有重复的行。
  ```
  $ uniq -d mm_gene_names.txt | wc -l
  ```
### Join
+ The Unix tool join is used to join different files together by a common column.
  可以使用echo $?来查看我们命令运行的返回码，根据返回码可以判断命令运行情况。
+ join的用法：-1 <file_1_field> -2 <file_2_field> <file_1> <file_2>。注意，join之前需要
  先排序。
  ```
  example.bed原始数据
  chr1 26 39
  chr1 32 47
  chr3 11 28
  chr1 40 49
  chr3 16 27
  chr1 9 28
  chr2 35 54
  chr1 10 19
  example_lengths.txt原始数据
  chr1 58352
  chr2 39521
  chr3 24859
  先对要join的列进行排序
  $ sort -k1,1 example.bed > example_sorted.bed
  $ sort -c -k1,1 example_lengths.txt
  $ join -1 1 -2 1 example_sorted.bed example_lengths.txt > example_with_lengths.txt
  $ cat example_with_lengths.txt
  chr1 10 19 58352
  chr1 26 39 58352
  chr1 32 47 58352
  chr1 40 49 58352
  chr1 9 28 58352
  chr2 35 54 39521
  chr3 11 28 24859
  chr3 16 27 24859
  然后就要检查join是否成功，分别统计join的第一个文件的行数以及join结果文件的行数
  $ wc -l example_sorted.bed example_with_lengths.txt
  ```
+ join加上-a option可以将join中不配对的行也输出到结果中。形成如下的结果
  ```
  chr1 10 19 58352
  chr1 26 39 58352
  chr1 32 47 58352
  chr1 40 49 58352
  chr1 9 28 58352
  chr2 35 54 39521
  chr3 11 28
  chr3 16 27
  ```
### Text Processing with Awk
+ 前面我们介绍的grep，cut，join，sort对于简单的任务可以胜任，但是对于稍显复杂的任务就需要awk了。
+ awk被设计为用来处理表数据，所以awk处理的是record，一般指的是表中的行，awk的结构为 pattern
  {action},pattern是一个expression或者regular expression pattern。如果pattern中的表达式
  为真或者regex匹配，那么就执行action。
  ```
  $ awk '{print $0}' example.bed 打印example中的所有record，这条语句没有pattern，就会直接
  执行action了。其中$0代表整个记录，而$1代表记录的第一个field，以此类推。
  或者
  $ awk '{print $2 "\t" $3}' example.bed
  26 39
  32 47
  11 28
  40 49
  16 27
  9  28
  35 54
  10 19
  $ awk '$3-$2>18' example.bed
  chr1  9  28
  chr2  35 24
  这里只用到了pattern，输出第三个field和第二个field相差18的record。
  $ awk '$1 ~ /chr1/ && $3-$2>10' example.bed
  这里的~代表第一个field遵守后面的正则表达式，并且满足后面的大小关系，然后会把符合的结果输出。
  或者
  $ awk '$1 ~ /chr2|chr3/ {print $0 "\t" $3-$2}' example.bed
  chr3 11 28 17
  chr3 16 27 11
  chr2 35 54 19
  这个命令既有pattern也有action，首先找出满足pattern的record，然后执行Acton中的语句输出。
  ```
  所以可以看出awk主要在两个时候会被用到
  1. 当我们过滤数据的规则中既包含了正则表达式有包含了算术规则的时候。
  2. 当我们需要使用算术规则重新定义数据的列的格式的时候。
+ 在awk中我们也可以使用begin和end来写命令语句
  ```
  $ awk 'BEGIN{s=0}; {s+=($3-$2)}; END{print "mean: " s/NR};' example.bed
  mean: 14
  这里我们用到了awk中的另一个常量NR，NR就是current record number。
  BEgin语句在我们读取第一条记录之前运行，而END语句在读取完最后一条语句之后运行。这条语句首先
  定义了一个变量初始化为0，然后读取每一条记录的时候运行中间的action，最后读取完所有的记录后输出
  s/NR。
  $ awk 'NR>=3 && NR<=5' example.bed
  输出文件中第三行和第五行之间的数据。
  ```
+ While Awk is designed to work with whitespace-separated tabular data, it’s easy to set   
  a different field separator: simply specify which separator to use with the -F argument.
  For example, we could work with a CSV file in Awk by starting with awk -F",".
+ 我们也可以使用awk轻松的转换各种生物信息学中的文件比如说 BED和GTF。
  ```
  $ awk '!/^#/ {print $1 "\t" $4-1 "\t" $5}' Mus_musculus.GRCm38.75_chr1.gtf | head -n3
  1 3054232 3054733
  1 3054232 3054733
  1 3054232 3054733
  ```
+ awk中也定义一种关联数组，即associative array。有点类型python中的dict。
  ```
  $awk '/Lypla1/ {feature[$3]+=1}; END {for (k in feature) print k "\t" feature[k]}'   Mus_musculus.GRCm38.75_chr1.gtf
  这条语句首先找到名为Lypla1的基因，然后将这个基因的第三个filed作为关联数组feature的索引值，
  每次碰到相同的field，feature索引值对应的数字就加一。最后读完所有的record后输出基因的第三个
  field以及这个field出现的次数。
  要达到相同的目的，使用unix命令行的方式也可以。
  $ grep 'Lypla1' Mus_musculus.GRCm38.75_chr1.gtf | cut -f3 | sort | uniq -c
  ```
  可以看出，awk的确是一种编程语言，有时候如果我们写太长的awk语句，那么也可以使用python代替。
### Bioawk:An Awk for Biological Formats
+
### Stream Editing with Sed
## Advanced Shell Tricks
### Subshells
### Named Pipes and Process Substitution
## The Unix Philosophy Revisited
