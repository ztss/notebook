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
+ sed reads data from a file or standard input and can edit a line at a time.
+ sed替换选项
  ```
  $ sed 's/pattern/replacement/' text.txt
  $ sed 's/chrom/chr/' chroms.txt | head -n 3
  ```
## Advanced Shell Tricks
### Subshells
### Named Pipes and Process Substitution
+ First in first out.在我们使用pipe的时候，可能需要同时输入输出多个文件。这个时候我们可以创建named
  file。
  ```
  $ mkfifo fqin
  $ ls -l fqin
  prw-r--r-1 vinceb staff
  我们创建了一个名叫fqin的named pipe。
  ```
+ 有时候单独的pipe我们不能处理这种情况
  ```
  $ processing_tool --in1 in1.fq --in2 in2.fq --out1 out2.fq --out2.fq
  这个时候，in1.fq,in2.fq,out1.fq,out2.fq都可以是named pipe。
  ```
  我们可以使用一种叫做process Substitution的方法来隐式的定义named pipe。
  ```
  $ rm fqin
  $ cat <(echo "hello, process substitution")
  Your shell then replaces this chunk (the <(...) part) with the path to this anonymous named pipe.
  改写上面的命令
  $ program --in1 <(makein raw1.txt) --in2 <(makein raw2.txt) --out1 out1.txt --out2 out2.txt
  或者
  $ program --in1 in1.txt --in2 in2.txt  --out1 >(gzip > out1.txt.gz) --out2 >(gzip > out2.txt.gz)
  这样就创建了两个anonymous named pipes。
  ```
## The Unix Philosophy Revisited



# A Rapid Introduction to the R Language
+ R通常是作为一种处理数据的编程语言。画图在生物信息学中是非常重要的，它可能揭示一些统计学模型
  或者假设检验中没有的模式。
+ 本章的目的是教您EDA(exploratory data analysis)技能，使您可以在任何分析阶段自由探索和试验您的数据。
## Geting Started with R and RStudio
+ 我们必须将我们数据分析过程中所写的代码存入R script中。
+ The Comprehensive R Archive Network(CRAN)。可以用install.packages("")下载到大多数R包。
## R Language Basics
### Simple Calculations in R, Calling Functions, and Getting Help in R
+ R默认打印七位有效数字。也可以使用
  ```
  print(sqrt(3.5),digits=10)
  打印10为有效数字
  可以用 getOption('digits')获得打印数位的全局选项，也可以使用
  options(digits=9)来将默认数位选项改为9位。
  > round(sqrt(3.5), digits=3)
  [1] 1.871
  通过在函数中指定digits，可以指定函数作用到参数第几个数位。
  ```
+ 因为R中含有太多的函数，所以我们在使用R的时候必须掌握两点
  1. 学会看函数的文档了解他的参数和他的原理。(?function or help(function))
  2. 能够找到新的有用的函数。(??"函数的简短描述")
### Variables and Assignment
+ R中也可以使用=好代替<-，可以用ls()看R中我们赋值的变量。
### Vectors, Vectorization, and Indexing
+ 在R中可以用c()函数来将很多值组合成list或者vector。
+ 在R中vector十分重要，因为它可以允许我们不写loop就能遍历整个vector，而且r中的算术运算符也
  可以直接对vector使用。
+
  ```
  > seq(3, 5)
  [1] 3 4 5
  > 1:5
  [1] 1 2 3 4 5
  > c(1, 2) + c(0, 0, 0, 0)
  [1] 1 2 1 2
  > c(1, 2) + c(0, 0, 0)
  [1] 1 2 1
  上面两条语句说明了R中的一个机制recycling。
  ```
+ R中的许多函数也可以直接对vector使用。通常R利用这种向量化来避免loop，因为使用vector而不是
  明确的loop可以明显的加快运算速度。
+ R中的vector也可以有名字。
  ```
  > b <- c(a=3.4, b=5.4, c=0.4)
  > b
    a b c
    3.4 5.4 0.4
  > names(b)
  [1] "a" "b" "c"
  > names(b) <- c("x", "y", "z") # change these names
  > b
    x y z
    3.4 5.4 0.4
  这个时候可以使用名字直接访问vector中的元素。
  比如说b['x']
  ```
+ 在R中也可以直接从一个vector中取出多个元素。这是R作为一种数据操作语言的一种非常好的特性。
  ```
  > x[c(2,3)]
  [1] 95.3 0.4
  ```
  这种特性称为矢量化索引，是一种操作数据的十分强大的方法。
  R中对于超过索引值的访问不会报错，而是会直接给出一个NA值。
  ```
  > z <- c(3.4, 2.2, 0.4, -0.4, 1.2)
  > z[c(2, 1, 10)]
  [1] 2.2 3.4 NA
  ```
  在R中使用负值索引可以排除这个索引值绝对值部分的值，打印出其他的所有值。
  ```
  > z[c(-4, -5)]
  [1] 3.4 2.2 0.4
  ```
  索引能够用来将vector重新排序
  ```
  > z[5:1]
  [1] 1.2 -0.4 0.4 2.2 3.4
  这个是逆向排列
  ```
  我们也可以使用R中的函数来或者一个向量的正向排序序列索引。
  ```
  > order(z)
  [1] 4 3 5 2 1
  > z[order(z)]
  [1] -0.4 0.4 1.2 2.2 3.4
  > order(z, decreasing=TRUE)
  [1] 1 2 5 3 4
  > z[order(z, decreasing=TRUE)]
  [1] 3.4 2.2 1.2 0.4 -0.4
  使用decreasing选项可以得到序列的逆序索引
  ```
  也可以使用运算符来构造一个vector的logical索引值
  ```
  > v <- c(2.3, 6, -3, 3.8, 2, -1.1)
  > v <= -3
  [1] FALSE FALSE TRUE FALSE FALSE FALSE
  使用这个功能我们可以取出一个vector中所有大于2的值
  > v[v > 2]
  [1] 2.3 6.0 3.8
  > v > 2 & v < 4
  [1] TRUE FALSE FALSE TRUE FALSE FALSE
  > v[v > 2 & v < 4]
  [1] 2.3 3.8
  ```
#### Vector types
+ R中的vector种类
  1. Numeric
  2. Integer
  3. Character
  4. Logical
+ R中的四种特殊值
  1. NA
  2. NULL
  3. -Inf,Inf
  4. NaN
+ R中对象的转换的本质是不能在转换过程中损失信息。
#### Factors and classes in R
+ R中还有一种vector叫做factors，可以这样建立
  ```
  > chr_hits <- c("chr2", "chr2", "chr3", "chrX", "chr2", "chr3", "chr3")
  > hits <- factor(chr_hits)
  > hits
  [1] chr2 chr2 chr3 chrX chr2 chr3 chr3
  Levels: chr2 chr3 chrX
  可以看到factors中不仅有原始的序列，还有一个levels,这里面包含了不同的固定的值。
  > levels(hits)
  [1] "chr2" "chr3" "chrX"
  > hits <- factor(chr_hits, levels=c("chrX", "chrY", "chr2", "chr3", "chr4"))
  > hits
  [1] chr2 chr2 chr3 chrX chr2 chr3 chr3
  Levels: chrX chrY chr2 chr3 chr4
  上面的factors创建语句将所有的基因都包含进去了，虽然序列中并没有一些基因。
  ```
  可以统计factors中每一个levels的个数
  ```
  > table(hits)
  hits
  chrX chrY chr2 chr3 chr4
  1    0    3    3    0
  ```
## Working with and Visualizing Data in R
### Loading Data into R
+ R中的路径操作
  ```
  > getwd() 获得当前的工作路径
  > setwd("path") 设置自己的工作路径
  ```
+ 有两个方法可以提升R读入大文件时候的效率
  1. First, we could explicitly set the class of each column through the colClasses argument.
  2. Additionally, specifying how many rows there are in your data by setting nrow in  read.delim() can lead to some performance gains.
  3. R的数据读入函数也可以读压缩文件，所以可以把大文件先压缩在读入R中。
+ 读入数据操作，read.csv(),read.delim()分别用来读入CSV文件和tab-delimited文件。
  ```
  > d <- read.csv("Dataset_S1.txt")
  read.csv()和read.delim()函数的header参数默认为真，因为上面读入的文件第一行是列的名字而不是
  数据。所以采用默认值。
  而对于没有列名的文件我们也可以赋予他们列名
  > bd <- read.delim("noheader.bed", header=FALSE, col.names=c("chrom", "start", "end"))
  ```
+ 通常，读入R中的数据有两种类型
    1. wide format:每一个测量变量都有自己的列。
    2. long format:一列用于存储测量的变量类型，另一列用于存储测量值。
  许多中情况下，由人类排列的数据都是wide format，但是当我们想要画图的时候，却需要的是long format。
  使用reshape2包中的函数，melt()可以将wide data变为long data。cast则相反。
### Exploring and Transforming Dataframes
+ 这个时候由R读入的数据是dataframe。数据帧是R用于存储表格数据的主力数据结构。dataframe中的每一列
  都是一个vector，即每一列都是相同类型的数据。storing columns of heterogeneous types of vectors is what dataframes are designed to do.
  ```
  > head(d,n=3)查看读入文件的前三行。
  > nrow(d)
  [1] 59140
  > ncol(d)
  [1] 16
  > dim(d)
  1] 59140 16
  查看读入文件的维度。
  > colnames(d)
  查看读入文件的列名。
  > colnames(d)[12] <- "percent.GC"
  改变列名
  ```
+ 我们也可以将一系列的vector创建为data.frame
  ```
  > x <- sample(1:50, 300, replace=TRUE)
  > y <- 3.2*x + rnorm(300, 0, 40)
  > d_sim <- data.frame(y=y, x=x)
  ```
+ 使用d$depth 可以访问dataframe中的一个列。使用df[row,col]可以取得dataframe中指定row和col的数据。
  也可以使用列名去访问。
  ```
  > d[1, c("start", "end")]
    start end
  1 55001 56000
  通常取得dataframe中的一列的输出是vector，但是如果我们想取得一列的结果仍然是dataframe。可以这样
  > d[, "start", drop=FALSE]
  ```
  我们也可以在我们读入的dataframe数据中增加一列。
  ```
  > d$cent <- d$start >= 25800000 & d$end <= 29700000
  > table(d$cent)
  FALSE TRUE
  58455 685
  ```
### Exploring Data Through Slicing and Dicing: Subsetting Dataframes
+ 我们能够通过逻辑向量来筛选出符合我们要求的行。
  ```
  > d$total.SNPs >= 85
  [1] FALSE FALSE FALSE FALSE FALSE FALSE [...]
  d中total.SNPS大于等于85的列为真。所以
  > d[d$total.SNPs >= 85, ]
  可以筛选出满足条件的行。
  也可以将几个逻辑表达式结合起来
  > d[d$Pi > 16 & d$percent.GC > 80, ]
  当然也可以指定选出特定的列
  > d[d$Pi > 16 & d$percent.GC > 80, c("start", "end", "depth", "Pi")]
  or
  > d[d$Pi > 16 & d$percent.GC > 80, c("start", "end", "Pi", "depth")]
  dataframe中每列都是一个vector，所以如果只想在特定的某列中筛选，可以这样
  > d$percent.GC[d$Pi > 16]
  或者我们想输出最开始的4个Pi值大于10的行号，如下
  > which(d$Pi > 10)[1:4]
  [1] 2 16 21 23
  也可以用subset函数达到相同的目的
  > subset(d, Pi > 16 & percent.GC > 80, c(start, end, Pi, percent.GC, depth))
  ```
### Exploring Data Visually with ggplot2 I: Scatterplots and Densities
+ 使用ggplot2快速的画图。
  ```
  library(ggplot2)
  > d$position <- (d$end + d$start) / 2
  > ggplot(d) + geom_point(aes(x=position, y=diversity))
  上面画图的代码分为两步，首先使用ggplot(d)将dataframe应用到plot上。ggplot2只能应用于
  dataframe。所以需要将数据转换为dataframe才能使用ggplot2画图。然后后面的加号为plot添加
  layer。layer可以分为散点图或者线图等等。上面的geom_point即添加一个散点图的layer。也称
  geom_point为geometric object。每一个object都有不同的aesthetic attributes。上面的aes函数
  是将dataframe中的列映射到aesthetic attributes的函数。
  > ggplot(d, aes(x=position, y=diversity)) + geom_point() 与上面的语句一样。
  为了缓解过度绘制，可以使用一个transparency level即alpha。
  > ggplot(d) + geom_point(aes(x=position, y=diversity), alpha=0.01)
  使用ggplot2画密度图
  > ggplot(d) + geom_density(aes(x=diversity), fill="black")
  这里的fill应用于所有的diversity点
  > ggplot(d) + geom_density(aes(x=diversity, fill=cent), alpha=0.4)
  这里的fill分为两部分，一部分cent=true，另一部分为cent=false。并且用alpha参数来区分这两个部分。
  ```
### Exploring Data Visually with ggplot2 II: Smoothing
+ 可视化是观察大型数据的一种很好的方法。
+ 在画散点图的时候，我们经常叠加一个smooth curve来看两个变量之间的关系。
  ```
  > ggplot(d, aes(x=depth, y=total.SNPs)) + geom_point() + geom_smooth()
  ```
### Binning Data with cut() and Bar Plots with ggplot2
+ 我们从复杂数据集中提取信息的另一种方法是通过分级（或离散化）来降低数据的分辨率。我们使用cut()函数
  来为数据分区间，如下
  ```
  > d$GC.binned <- cut(d$percent.GC, 5)
  将所选择的数据分为五个区间，新产生的列为factor，level是划分出的5个区间。可以用
  > table(d$GC.binned)
  (0.716,17.7] (17.7,34.7] (34.7,51.6] (51.6,68.5] (68.5,85.6]
    6            4976         45784       8122          252
  来查看落在5个区间中的数字的个数。
  也可以认为的划分区间
  > cut(d$percent.GC, c(0, 25, 50, 75, 100))
  根据上面的区间可以画出条状图
  > ggplot(d) + geom_bar(aes(x=GC.binned))
  直接使用geom_bar会自动对所选的数据分区间。
  也可以根据所选的区间来为变量画出密度图
  > ggplot(d) + geom_density(aes(x=depth, linetype=GC.binned), alpha=0.5)
  使用以下语句可以选择不同的划分区间大小，从而找到最合适的能够体现数据特征的binwidth。
  > ggplot(d) + geom_bar(aes(x=Pi), binwidth=1) + scale_x_continu ous(limits=c(0.01, 80)).
  ```
### Merging and Combining Data: Matching Vectors and Merging Dataframes
+ 将几个数据集集合起来分析也是一种很重要的数据分析的能力。
+ 可以使用%in%来找出一个vector中的元素是否在后面的那个vector中。
  ```
  > c(3, 4, -1) %in% c(1, 3, 4, 8)
  [1] TRUE TRUE FALSE
  读入数据
  > reps <- read.delim("chrX_rmsk.txt.gz", header=TRUE)
  我们构造一个vector
  > common_repclass <- c("SINE", "LINE", "LTR", "DNA", "Simple_repeat")
  我们可以挑选出repClass在上面这个vector的行
  > reps[reps$repClass %in% common_repclass, ]
  我们可以使用如下语句挑选出repClass中数量最多的五个类
  > sort(table(reps$repClass), decreasing=TRUE)[1:5]
  > top5_repclass <- names(sort(table(reps$repClass), decreasing=TRUE)[1:5])
  match()函数也有%in%的功能
  > match(c("A", "C", "E", "A"), c("A", "B", "A", "E"))
  [1] 1 NA 4 1
  match返回第一个vector中元素在第二个vector中出现的第一次的索引。如果没有找到，则返回NA。
  match的输出可以用来join两个dataframe通过共同的column。
  ```
+ 读入数据
  ```
  > mtfs <- read.delim("motif_recombrates.txt", header=TRUE)
  > rpts <- read.delim("motif_repeats.txt", header=TRUE)
  为两个数据增加列
  mtfs$pos <- paste(mtfs$chr,mtfs$motif_start,sep="-")
  rpts$pos <- paste(rpts$chr,rpts$motif_start,sep="-")
  增加的这两列为两个数据集公有的列
  查看两个数据中公有的数据的条目
  > table(mtfs$pos %in% rpts$pos)
  FALSE TRUE
  10832 9218
  > i <- match(mtfs$pos, rpts$pos)
  使用上面的vector为rpts增加一个列
  > mtfs$repeat_name <- rpts$name[i]
  可以移除mtfs中repeat_name为空的行
  mtfs_inner <- mtfs[!is.na(mtfs$repeat_name),]
  merge两个数据集
  recm <- merge(mtfs,rpts,by.x="pos",by.y = "pos")
  可以看出，mtfs_inner和recm得出的值基本一样。所以merge函数是十分有用的。
  merge函数默认是 inner join。
  通过添加参数all.x=TRUE，可以实现left outer joins，通过添加all.y=TRUE，可以实现right outer join。

  ```
### Using ggplot2 Facets
+ 使用facet_wrap()和facet_grid()可以对一个dataframe画出多个图，其中每个图可以设置为一个列的不同level。
  ```
  > p <- ggplot(mtfs, aes(x=dist, y=recom)) + geom_point(size=1, color="grey")
  > p <- p + geom_smooth(method='loess', se=FALSE, span=1/16)
  > p <- p + facet_grid(repeat_name ~ motif)
  > print(p)
  ```
### More R Data Structures: Lists
+ 不同于前面所学的vector只能包含相同的数据类型，list可以包含不同的数据类型，并且list可以包含R中的任何
  类型，包括另一个list。dataframe是list。
  ```
  创建一个list
  > adh <- list(chr="2L", start=14615555L, end=14618902L, name="Adh")
  ```
+ 如果我们将不同的类型存在vector中，那么会进行变量强制转换，因为vector只能包含一种数据类型。
+ 访问list中的值
  ```
  > adh[1:2] $chr
  [1] "2L"
  $start
  [1]
  14615555
  使用str()可以打印出list的结构类型,即包含的嵌套结构。
  > z <- list(a=list(rn1=rnorm(20), rn2=rnorm(20)), b=rnorm(10))
  > str(z)
  List of 2
   $ a:List of 2
    ..$ rn1: num [1:20] -2.8126 1.0328 -0.6777 0.0821 0.7532 ...
    ..$ rn2: num [1:20] 1.09 1.27 1.31 2.03 -1.05 ...
  $ b: num [1:10] 0.571 0.929 1.494 1.123 1.713 ...
  > adh[[2]]
  [1] 14615555
  > adh[['start']]
  [1] 14615555
  也可以如此访问。或者
  > adh$chr
  可以使用names()输出list的名字
  > names(adh)
  [1] "chr"   "start" "end"   "name"
  ```
+ dataframes are built from lists, and each dataframe column is a vector stored as a
  list element.
### Writing and Applying Functions to Lists with lapply() and sapply()
#### Using lapply()
+ 创建实例数据
  ```
  > ll <- list(a=rnorm(6, mean=1), b=rnorm(6, mean=4), c=rnorm(6, mean=6))
  假如我们想要求解每一个list中vector的mean,可以使用lapply()来将一个函数应用于list上。
  > lapply(ll, mean)
  $a
  [1] 0.967971

  $b
  [1] 3.859597

  $c
  [1] 5.861516
  通常这个方法可以让我们避免显式的使用loop。
  如果list中的vector中有一个元素为NA,那么就要使用下面语句避免lapply()返回NA值
  > lapply(ll, mean, na.rm=TRUE)
  ```
+ 利用此函数可以简单的实现并行运算
  ```
  > library(parallel)
  > results <- mclapply(my_samples,slowFunction)
  也可以设置并行运算的核数
  > options(cores=3)
  > getOption('cores')
  [1] 3
  ```
#### Writing functions
+ 可以自己写函数来应用于lapply()。
  ```
  一般函数的格式如图
  fun_name <- function(args) {
    # body, containing R expressions
    return(value)
  }
  例如
  > meanRemoveNA <- function(x) mean(x, na.rm=TRUE)
  > lapply(ll, meanRemoveNA)
  ```
#### Digression: Debugging R Code
+ 函数是一种很好的功能，但是也很难debugging。但是掌握debug技巧是非常重要的。browser()是一种设置断点
  的技巧。可以用于debug。
  ```
  function(x){
   a <- 2
   browser()
   y <- x+a
   return(y)
  }
  运行这个函数会让我们进入debug界面。在R的命令输入行输入ls()可以打印当前所有的变量,输入n可以执行下一行
  ，输入c可以继续运行代码，输入Q可以退出并且继续运行代码。
  ```
#### More list apply functions: sapply() and mapply()
+ sapply()跟lapply()差不多，但是它返回的结果更加简单。
  ```
  > sapply(ll, function(x) mean(x,na.rm = TRUE))
        a         b         c
  0.7203142 3.8595975 5.8615159
  ```
+ mapply()是sapply()的多变量版本，用法如下
  ```
  > ind_1 <- list(loci_1=c("T", "T"), loci_2=c("T", "G"), loci_3=c("C", "G"))
  > ind_2 <- list(loci_1=c("A", "A"), loci_2=c("G", "G"), loci_3=c("C", "G"))
  > mapply(function(a,b) length(intersect(a,b)), ind_1,ind_2)
  loci_1 loci_2 loci_3
     0      1      2
  ```
+ 在R中还有matries,和arrays这两种数据结构，他们都是多维的R vector。我们通常是使用dataframe而不是这
  两种数据类型，因为dataframe可以包含不同数据类型的列。他们也有自己的apply function。apply(),sweep()。
### Working with the Split-Apply-Combine Pattern
+ Grouping data is a powerful method in exploratory data analysis.
+ 在这一节中，我们将学习如何使用通用的数据分析模式去分类数据，并且在每一个类上应用函数，然后将这些
  结果结合起来。这个模式的名字叫做 split-apply-combine。
+ 过程
  ```
  将数据分开
  d_split <- split(d$depth,d$GC.binned)
  这个语句将d$depth这个vector按照GC.binned分开。并且返回一个list。
  然后，对于这样的每个group就可以使用lapply()将一个函数应用于每一个group上。
  > lapply(d_split, mean)
  这一步也可以使用其他的方法，将结果输出为一个vector，如下两种
  > sapply(d_split, mean)
  > unlist(lapply(d_split, mean))
  也可以将数据分开后，同时应用apply
  > dpth_summ <- lapply(split(d$depth, d$GC.binned), summary)
  最后，就是将这些结果整合到一起，同时使用do.call()和rbind
  > do.call(rbind, lapply(split(d$depth, d$GC.binned), summary))
  ```
+ split()函数有一些特性
  1. First, it’s possible to group by more than one factor—just provide split() with a list of factors. split() will split the data by all combinations of these factors.
  2. Second, you can unsplit a list back into its original vectors using the function unsplit().
  3. although we split single columns of a dataframe (which are just vectors), split() will happily split dataframes. (将dataframe分类是非常有必要的，当我们的apply部分需要用到多个columns).
+ R中也有一些函数将上面的模式结合在一起，只需要一个函数即可。如下
  ```
  > tapply(d$depth, d$GC.binned, mean)
  (0.716,17.7]  (17.7,34.7]  (34.7,51.6]  (51.6,68.5]  (68.5,85.6]
    3.810000     8.788244     8.296699     7.309941     4.037698
  > aggregate(d$depth, list(gc=d$GC.binned), mean)
           gc        x
  1 (0.716,17.7] 3.810000
  2  (17.7,34.7] 8.788244
  3  (34.7,51.6] 8.296699
  4  (51.6,68.5] 7.309941
  5  (68.5,85.6] 4.037698
  ```
+ The take-home point of this section: the split-apply-combine pattern is an essential part of  data analysis.
### Exploring Dataframes with dplyr
+ dplyr包是专门处理dataframe数据类型的。速度非常快，因为他是用C++写的。其中有五个基础的函数用来操作
  dataframe，arrange(), filter(), mutate(), select(), and summarize()。
+ 首先将dataframe转换为dplyr特有的tbl_df类型。
  ```
  > library(dplyr)
  > d_df <- tbl_df(d)
  从tbl_df类中挑选几列
  > select(d_df, start, end, Pi, Recombination, depth)
  也可以这样使用
  > select(d_df, -(start:cent))
  这样的话挑选出除了start和cent范围之外的所有列
  也能使用filter()函数对数据的行进行筛选
  > filter(d_df, Pi > 16, percent.GC > 80)
  也能使用arrange函数对列进行排序。与d[order(d$percent.GC), ]功能相似
  > arrange(d_df, depth)
  也能通过集合desc函数来使用arrange函数进行降序排序
  > arrange(d_df, desc(total.SNPs), desc(depth))
  也能使用mutate函数来为dataframe添加新的列
  > d_df <- select(d_df, -diversity) # remove our earlier diversity column
  > d_df <- mutate(d_df, diversity = Pi/(10*1000))
  ```
+ 通常使用dplyr包的时候，我们喜欢将这个包的一系列操作写为一个pipeline，就跟linux里的pipeline一样
  只不过他的中间符号是%>%，其性质也与Linux中差不多。如下
  ```
  > d_df %>% mutate(GC.scaled=scale(percent.GC)) %>%
  filter(GC.scaled>4,depth>4) %>%
  select(start,end,depth,GC.scaled,percent.GC) %>%
  arrange(desc(depth))
  ```
+ dplyr’s raw power comes from the way it handles grouping and summarizing data.
  ```
  > mtfs_df <- tbl_df(mtfs)
  来给上面的tbl_df分类
  > mtfs_df %>% group_by(chr)
  然后对分类后的数据进行summary
  > mtfs_df %>%
      group_by(chr) %>%
      summarize(max_recom = max(recom), mean_recom = mean(recom), num=n())
  ```
+ dplyr最好的特征是上面的方法能够直接在数据库连接中使用。
### Working with Strings
+ R中所有的string都是character vector。
  ```
  使用nchar()查看string中字符的个数
  使用grep(pattern, x)查找包含pattern的元素在x中的位置。
  R中的grep可以使用Perl。如下
  > chrs <- c("chrom6", "chr2", "chr6", "chr4", "chr1", "chr16", " chrom8")
  > chrs[grep("[^\\d]6", chrs, perl=TRUE)]
  [1] "chrom6" "chr6"
  regexpr(pattern, x) returns where in each element of x it matched pattern.
  > regexpr("[^\\d]6",chrs,perl = TRUE)
  [1]  5 -1  3 -1 -1 -1 -1
  attr(,"match.length")
  [1]  2 -1  2 -1 -1 -1 -1
  attr(,"index.type")
  [1] "chars"
  attr(,"useBytes")
  [1] TRUE
  我们可以利用下面的语句来整理chrs
  > pos <- regexpr("\\d+",chrs,perl = TRUE)
  [1] 6 4 4 4 4 4 7
  attr(,"match.length")
  [1] 1 1 1 1 1 2 1
  attr(,"index.type")
  [1] "chars"
  attr(,"useBytes")
  [1] TRUE
  > substr(chrs,pos,pos+attributes(pos)$match.length)
  [1] "6"  "2"  "6"  "4"  "1"  "16" "8"
  > sub(pattern="Watson", replacement="Watson, Franklin,", x="Watson and Crick discovered DNA's structure.")
  [1] "Watson, Franklin, and Crick discovered DNA's structure."
  ```
+ 在生物信息学中，解析不一致的命名是一项很需要花费时间的事情。
  ```
  使用strsplit()分隔字符串
  > leafy <- "gene=LEAFY;locus=2159208;gene_model=AT5G61850.1"
  > strsplit(leafy, ";")
  [[1]]
  [1] "gene=LEAFY" "locus=2159208" "gene_model=AT5G61850.1"
  ```
## Developing Workflows with R Scripts
### Control Flow: if, for, and while
+ 前面说过，使用apply()系列的函数可以避免在R中使用loop，但是还是有一些情况不得不使用循环控制
  语句。
  ```
  在for循环中，通常是创建一个索引Vector。
  for (i in 1:length(vec)) {
    # do something
  }
  ```
### Working with R Scripts
+ 可以使用source("my_rscript.R")来运行R script。
+ 使用sessioninfo()来查看R的版本以及每一个包的版本。
+ 也可以在命令行使用以下语句运行R脚本
  ```
  $ Rscript --vanilla my_rs.R
  ```
### Workflows for Loading and Combining Multiple Files
+ 这一节我们学习如何loading和组合多个文件。使用这一功能需要用到我们之前学习到的一些函数：
  apply(), do.call(), rbind(),sub().
  ```
  首先列出所有符合我们需要处理的文件
  > list.files("hotspots",pattern = "hotspots.*\\.bed")
  这里使用了正则表达式。
  > hs_files <- list.files("hotspots", pattern="hotspots.*\\.bed", full.names=TRUE)
  然后可以使用lapply()函数来处理着一些的文件
  > bedcols <- c("chr", "start", "end")
  > loadFile <- function(x) read.delim(x, header=FALSE, col.names=bedcols)
  > hs <- lapply(hs_files, loadFile)
  然后给hs的每一个list item取名字
  > names(hs) <- list.files("hotspots", pattern="hotspots.*\\.bed")
  然后将这些数据全部结合在一起。
  > hsd <- do.call(rbind, hs)
  这个时候hsd中的数据有行名，我们可以去掉行名
  > row.names(hsd) <- NULL
  ```
+ 我们也可以改变上面的loadFile函数。
  ```
  loadFile <- function(x) {
    # read in a BED file, extract the chromosome name from the file,
    # and add it as a column
    df <- read.delim(x, header=FALSE, col.names=bedcols)
    df$chr_name <- sub("hotspots_([^\\.]+)\\.bed", "\\1", basename(x))
    使用basename(x)来取出非文件路径的文件名字
    df$file <- x
    df
  }
  或者这样改写loadFile
  loadAndSummarizeFile <- function(x) {
    df <- read.table(x, header=FALSE, col.names=bedcols)
    data.frame(chr=unique(df$chr), n=nrow(df), mean_len=mean(df$end - df$start))
  }
  ```
### Exporting Data
+ 可以使用write.table()将dataframe导出为纯文本文件。
  ```
  > write.table(mtfs, file="hotspot_motifs.txt", quote=FALSE, sep="\t", row.names=FALSE, col.names=TRUE)
  也可以先把文件压缩然后再写出
  > hs_gzf <- gzfile("hotspot_motifs.txt.gz")
  > write.table(mtfs, file=hs_gzf, quote=FALSE, sep="\t", row.names=FALSE, col.names=TRUE)
  ```
  但是R中也有一些对象不适合导出为纯文本文件。这个时候可以导出为Rdata文件
  ```
  > tmp <- list(vec=rnorm(4), df=data.frame(a=1:3, b=3:5))
  > save(tmp, file="example.Rdata")
  > rm(tmp) # remove the original 'tmp' list
  > load("example.Rdata") # this fully restores the 'tmp' list from file
  > str(tmp)
  List of 2
  $ vec: num [1:4] -0.668 -0.279 -0.717 0.052
  $ df :'data.frame':	3 obs. of  2 variables:
  ..$ a: int [1:3] 1 2 3
  ..$ b: int [1:3] 3 4 5
  ```
## Further R Directions and Resources


# Working with Range Data
## A Crash Course in Genomic Ranges and Coordinate Systems
+ 为了指定基因组区域或者位置，我们需要一些信息。比如说染色体名字，range，每一个range都有一个起始位置
  以及结束位置，第三个就是strand，因为染色体DNA是double-stranded。所以我们需要直到一些特性是处于
  那个strand上面的。
+ range是位于染色体上的。
+ 不同的基因定位通常跟特定的参考基因组版本有关。所以也有一些软件提供重新map到新基因组版本的功能。
+ 有两种range System
  1. 0-based coordinate system, with half-closed, half-open intervals.这个系统中，sequence的第一个base是0，最后一个是-1。而且区间是[)这样的。
  python使用的就是这种。这种系统可以很好的表达0长度的features。
  2. 1-based coordinate system, with closed intervals.这个系统中，sequence的第一个为1，最后一个
  就是长度的大小，他的区间是形如[]这样的。R使用的是这种。Format/library BED GTF GFF SAM BAM VCF BCF Wiggle
  GenomicRanges BLAST GenBank/EMBL Feature Table Type 0-based 1-based 1-based 1-based 0-based 1-based
  0-based 1-based 1-based 1-based 1-based。这是一些文件对应的range type。
+ 几乎所有的范围格式，范围的坐标在参考序列的正向链上给出，这是因为DNA是double strand。例如，
  代表蛋白质编码区的范围仅在给定适当链的情况下才具有生物学意义。
## An Interactive Introduction to Range Data with Genomic Ranges
### Installing and Working with Bioconductor Packages
+ 使用包的语句
  ```
  > source("http://bioconductor.org/biocLite.R")
  > biocLite()
  ```
### Storing Generic Ranges with IRanges
+ 在处理生物学数据的时候，我们应该考虑如何引入range这个概念。
+ The purpose of the first part of this chapter is to teach you range thinking through the use
  of use Bioconductor’s IRanges package。下面是如何使用IRanges。
  ```
  > library(IRanges)
  可以使用这个包创建一个Irange对象。
  > rng <- IRanges(start=4, end=13)
  > rng
  IRanges of length 1
      start end width
  [1] 4 13 10
  也可以用vector来创建
  > x <- IRanges(start=c(4, 7, 2, 20), end=c(13, 7, 5, 23))
  > x
  IRanges object with 4 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]         4        13        10
  [2]         7         7         1
  [3]         2         5         4
  [4]        20        23         4
  也可以为range取个名字。
  > names(x) <- letters[1:4]
  使用以下语句查看类
  > str(x)
  Formal class 'IRanges' [package "IRanges"] with 6 slots
  ..@ start          : int [1:4] 4 7 2 20
  ..@ width          : int [1:4] 10 1 4 4
  ..@ NAMES          : chr [1:4] "a" "b" "c" "d"
  ..@ elementType    : chr "ANY"
  ..@ elementMetadata: NULL
  ..@ metadata       : list()
  ```
### Basic Range Operations: Arithmetic, Transformations, and Set Operations
+ IRanges对象可以使用算术运算符来scale。
  ```
  > x <- IRanges(start=c(40, 80), end=c(67, 114))
  > x + 4L
  也可以用一个bound来约束range的大小。
  > y <- IRanges(start=c(4, 6, 10, 12), width=13)
  > restrict(y,5,10)
  IRanges object with 3 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]         5        10         6
  [2]         6        10         5
  [3]        10        10         1
  可以看出只截取了在5到10范围的基因。
  另外还可以使用flank()来返回一个region边缘的range。
  > x
  IRanges object with 2 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]        40        67        28
  [2]        80       114        35
  > flank(x,width=7)
  IRanges object with 2 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]        33        39         7
  [2]        73        79         7
  通过设置start=false也可以取区域的另一边
  > flank(x,width=7,start=FALSE)
  IRanges object with 2 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]        68        74         7
  [2]       115       121         7
  还可以使用reduce函数将所有重叠的range整合在一起
  也可以使用gap()函数来返回range之间的gaps。
  IRanges对象也可以使用集合运算，即交集之类的。
  ```
### Finding Overlapping Ranges
+ 计算重叠在生物信息学中十分重要。一般我们使用findOverlaps()函数来寻找重叠。
  ```
  > qry <- IRanges(start=c(1, 26, 19, 11, 21, 7), end=c(16, 30, 19, 15, 24, 8), names=letters[1:6])
  > sbj <- IRanges(start=c(1, 19, 10), end=c(5, 29, 16), names=letters[24:26])
  > hts <- findOverlaps(qry,sbj)
  > names(qry)[queryHits(hts)]
  [1] "a" "a" "b" "c" "d" "e"
  > names(sbj)[subjectHits(hts)]
  [1] "x" "z" "y" "y" "z" "y"
  ```
  我们将重叠表示为query和subject之间的映射。这是一个多对多的映射，上面的hts中的列即为这个映射
  之间的对应关系。
  我们也可以只查找query整个被包含在subject中的重叠。
  ```
  > hts_within <- findOverlaps(qry, sbj, type="within")
  ```
  也可以在findOverlaps中添加select参数来选择返回第几个重叠匹配。
  ```
  > findOverlaps(qry, sbj, select="first")
  [1] 1 2 2 3 2 NA
  > findOverlaps(qry, sbj, select="last")
  [1] 3 2 2 3 2 NA
  > findOverlaps(qry, sbj, select="arbitrary")
  [1] 1 2 2 3 2 NA
  ```
+ 根据上面的分析，我们寻找query在subject上的重叠的复杂度是蛮高的，所以为了简化算法，我们使用
  interval tree这种数据结构
  ```
  我们可以使用IRange对象来创建IntervalTree，但是现有的版本使用NCList替代了原有的interval tree。
  > sbj_it <- NCList(sbj)
  NCList object with 3 ranges and 0 metadata columns:
        start       end     width
    <integer> <integer> <integer>
  x         1         5         5
  y        19        29        11
  z        10        16         7
  然后使用优化后的数据结构来使用findOverlaps
  > findOverlaps(qry,sbj_it)
  可以使用以下语句将结果转化为矩阵
  > as.matrix(hts)
  可以使用以下语句查看每一个query和多少个subject重叠了
  > countQueryHits(hts)
  [1] 2 1 1 1 1 0
  > setNames(countQueryHits(hts), names(qry))
  a b c d e f
  2 1 1 1 1 0
  也可以使用以下语句来查看有多少个subject和query重叠了
  > countSubjectHits(hts)
  使用ranges()函数输出每个重叠部分的坐标
  > ranges(hts, qry, sbj)
  > countOverlaps(qry, sbj)
  a b c d e f
  2 1 1 1 1 0
  > subsetByOverlaps(qry, sbj)
  IRanges object with 5 ranges and 0 metadata columns:
        start       end     width
    <integer> <integer> <integer>
  a         1        16        16
  b        26        30         5
  c        19        19         1
  d        11        15         5
  e        21        24         4
  上面的函数返回query上重叠的样本数目，只输出不同的。
  ```
### Finding Nearest Ranges and Calculating Distance
+ 这章主要介绍三个函数,nearest(),precede(),follow()。
  ```
  > qry <- IRanges(start=6, end=13, name='query')
  > sbj <- IRanges(start=c(2, 4, 18, 19), end=c(4, 5, 21, 24), names=1:4)
  nearest返回query离subject中最近的range索引。
  > nearest(qry, sbj)
  [1] 2
  precede返回query在subject中下一个range的索引，follow则是上一个
  > precede(qry, sbj)
  [1] 3
  > follow(qry, sbj)
  [1] 1
  query也可以是一系列的range。
  > qry2 <- IRanges(start=c(6, 7), width=3)
  > nearest(qry2, sbj)
  [1] 2 2
  ```
+ distance和distanceToNearest分别返回每对(一一对应)range之间的距离和query离subject中range的
  最近距离。
  ```
  qry <- IRanges(sample(seq_len(1000), 5), width=10)
  sbj <- IRanges(sample(seq_len(1000), 5), width=10)
  > distanceToNearest(qry, sbj)
  > distance(qry, sbj)
  ```
### Run Length Encoding and Views
+ 在这一节中，我们主要使用coverage data。
#### Run-length encoding and coverage()
+ 一般数据很大，我们首先需要压缩
  ```
  > x <- as.integer(c(4, 4, 4, 3, 3, 2, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 4, 4, 4, 4, 4, 4, 4))
  > xrle <- Rle(x)
  > xrle
  > xrle
  integer-Rle of length 28 with 7 runs
  Lengths: 3 2 1 5 7 3 7
  Values : 4 3 2 1 0 1 4
  > as.vector(xrle)
  [1] 4 4 4 3 3 2 1 1 1 1 1 0 0 0 0 0 0 0 1 1 1 4 4 4 4 4 4 4
  上面的Rle类支持这些操作：subsetting, arithemetic and comparison operations, summary functions, and math functions。
  而可以使用以下函数访问Rle类的长度和值
  > runLength(xrle)
  [1] 3 2 1 5 7 3 7
  > runValue(xrle)
  [1] 4 3 2 1 0 1 4
  ```
+ 计算一系列range的coverage。
  ```
  > set.seed(0)
  > rngs <- IRanges(start=sample(seq_len(60), 10), width=7)
  > names(rngs)[9] <- "A" # label one range for examples later
  > rngs_cov <- coverage(rngs)
  > rngs_cov
  integer-Rle of length 63 with 18 runs
     Lengths: 11 4 3 3 1 6 4 2 5 2 7 2 3 3 1 3 2 1
     Values : 0 1 2 1 2 1 0 1 2 1 0 1 2 3 4 3 2 1
  ```
  coverage:For each position in the space underlying a set of ranges, counts the number of ranges that cover it.即计算坐标系上每一个position上有几个range。
  这里返回的coverage就是一个Rle类。
  ```
  > rngs_cov > 3 # where is coverage greater than 3?
  logical-Rle of length 63 with 3 runs
    Lengths:56 1 6
    Values : FALSE TRUE FALSE
  > rngs_cov[as.vector(rngs_cov) > 3] # extract the depths that are greater than 3
  integer-Rle of length 1 with 1 run
  Lengths: 1
  Values : 4
  也可以使用labels来访问Rle对象
  > rngs_cov[rngs['A']]
  integer-Rle of length 7 with 2 runs
  Lengths: 5 2
  Values : 2 1
  ```
#### Going from run-length encoded sequences to ranges with slice()
+ 可以用slice函数以Rle(run-length encoded)对象为参数构造ranges。
  ```
  > min_cov2 <- slice(rngs_cov,lower = 2)
  > min_cov2
  Views on a 66-length Rle subject

  views:
      start end width
  [1]     4   7     4 [2 2 2 2]
  [2]    18  20     3 [2 2 2]
  [3]    23  24     2 [2 2]
  [4]    39  40     2 [2 2]
  [5]    43  45     3 [2 2 2]
  [6]    60  63     4 [2 2 2 2]
  ```
#### Advanced IRanges: Views
+ 使用一些统计函数来表示我们使用slice函数输出得到的view object。
  ```
  > viewMeans(min_cov2)
  > viewMaxs(min_cov2)
  > viewApply(min_cov2, median)
  ```
+ 我们将我们的region分为一个一个等分的窗口，然后分别计算窗口中每一个position的coverage。
  步骤如下
  ```
  > rngs_cov
  integer-Rle of length 66 with 19 runs
  Lengths: 3 4 3 3 4 3 2 2 5 4 5 2 2 3 4 7 3 4 3
  Values : 1 2 1 0 1 2 1 2 1 0 1 2 1 2 1 0 1 2 1
  > length(rngs_cov)
  [1] 66
  > bwidth <- 5L
  > end <- bwidth * floor(length(rngs_cov) / bwidth)
  > windows <- IRanges(start=seq(1, end, bwidth), width=bwidth)
  > head(windows)
  IRanges object with 6 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]         1         5         5
  [2]         6        10         5
  [3]        11        15         5
  [4]        16        20         5
  [5]        21        25         5
  [6]        26        30         5
  > cov_by_wnd <- Views(rngs_cov, windows)
  > head(cov_by_wnd)
  Views on a 66-length Rle subject

  views:
    start end width
  [1]     1   5     5 [1 1 1 2 2]
  [2]     6  10     5 [2 2 1 1 1]
  [3]    11  15     5 [0 0 0 1 1]
  [4]    16  20     5 [1 1 2 2 2]
  [5]    21  25     5 [1 1 2 2 1]
  [6]    26  30     5 [1 1 1 1 0]
  ```
  接下来我们会介绍GenomicRanges，因为这个是在IRange上扩展形成的，所以我们上面所学习的应用于IRange
  上的功能也能直接应用于GRanges上。
### Storing Genomic Ranges with GenomicRanges
+ GenomicRanges包引进了一个新的Granges类，比IRanges类多了两个信息：sequence name，和 strand。
  ```
  创建一个GRanges对象
  library(GenomicRanges)
  > gr <- GRanges(seqname=c("chr1", "chr1", "chr2", "chr3"),ranges = IRanges(start = 5:8,width = 10),strand = c("+", "-", "-", "+"))
  > gr
  GRanges object with 4 ranges and 0 metadata columns:
      seqnames    ranges strand
         <Rle> <IRanges>  <Rle>
  [1]     chr1      5-14      +
  [2]     chr1      6-15      -
  [3]     chr2      7-16      -
  [4]     chr3      8-17      +
  -------
  seqinfo: 3 sequences from an unspecified genome; no seqlengths
  也可以为GRanges构造器指定sequence length。
  > seqlens <- c(chr1=152, chr2=432, chr3=903)
  > gr <- GRanges(seqname=c("chr1", "chr1", "chr2", "chr3"), ranges=IRanges(start=5:8, width=10), strand=c("+", "-", "-", "+"),gc=round(runif(4), 3), seqlengths=seqlens)
  可以使用下面的语句访问GRanges中的数据
  > start(gr)
  [1] 5 6 7 8
  > end(gr)
  [1] 14 15 16 17
  > width(gr)
  [1] 10 10 10 10
  > seqnames(gr)
  > strand(gr)
  上面两个函数返回的都是 run-length encoded 对象。如果我们想要提取GRanges中的IRanges对象
  > ranges(gr)
  可以查看GRanges的长度，以及为它命名
  > length(gr)
  [1] 4
  > names(gr) <- letters[1:length(gr)]
  GRanges也支持子集操作。
  > start(gr) > 7
  [1] FALSE FALSE FALSE TRUE
  > gr[start(gr) > 7]
  使用以下语句可以统计每个chromosome有多少个ranges
  > table(seqnames(gr))

  chr1 chr2 chr3
   2    1    1
  可以通过访问染色体名字取得每个染色体的GRanges的子集
  > gr[seqnames(gr) == "chr1"]
  使用以下语句可以访问metadata columns
  > mcols(gr)
  DataFrame with 4 rows and 1 column
       gc
    <numeric>
  a     0.206
  b     0.177
  c     0.687
  d     0.384
  上面返回的是dataframe，所以也可以使用$访问数据
  > mcols(gr)$gc
  [1] 0.897 0.266 0.372 0.573
  > gr$gc
  [1] 0.897 0.266 0.372 0.573
  可以计算chr1所有ranges的average GC content
  > mcols(gr[seqnames(gr) == "chr1"])$gc
  [1] 0.897 0.266
  > mean(mcols(gr[seqnames(gr) == "chr1"])$gc)
  [1] 0.5815
  ```
### Grouping Data with GRangesList
+ 使用GRangesList将数据分组
  ```
  创建GRangesList
  > gr1 <- GRanges(c("chr1", "chr2"), IRanges(start=c(32, 95), width=c(24, 123)))
  > gr2 <- GRanges(c("chr8", "chr2"), IRanges(start=c(27, 12), width=c(42, 34)))
  > grl <- GRangesList(gr1, gr2)
  ```
  GRangesList于list的性质类似。
  使用unlist可以将GRangesList中的元素结合到一个单独的GRanges对象中(much like unlisting an R list of vectors to create one long vector)。
  ```
  > unlist(grl)
  ```
  也能使用c()函数将多个GRangesList对象组合起来
  跟访问list对象一样，也可以使用list element names函数来访问GRangesList对象中的features。
  在R中有很多由不同对象组成的list,他们的性质都与原始的list差不多。
  通过seqnames对GRanges对象分组
  ```
  > chrs <- c("chr3", "chr1", "chr2", "chr2", "chr3", "chr1")
  > gr <- GRanges(chrs, IRanges(sample(1:100, 6, replace=TRUE), width=sample(3:30, 6, replace=TRUE)))
  > gr_split <- split(gr, seqnames(gr))
  > names(gr_split)
  [1] "chr3" "chr1" "chr2"
  得到的gr_split是GRangesList对象
  ```
  也可以使用下列语句还原
  ```
  > unsplit(gr_split, seqnames(gr))
  ```
  上面将GRanges对象变为不同组的GRangesList对象就可以将函数应用于不同的分组上。即使用lapply()或者
  sapply()函数
  ```
  > lapply(gr_split, function(x) order(width(x)))
  > sapply(gr_split, function(x) min(start(x)))
  > sapply(gr_split, length)
  ```
  也可以直接将overlap函数应用于GRangesList上
  ```
  > reduce(gr_split)
  ```
  这说明GRangesList对象的一个重要的性质：many methods applied to GRangesList objects work at the grouped-data level automatically。
### Working with Annotation Data: GenomicFeatures and rtracklayer
+ 这一小节介绍两个package。
     1. GenomicFeatures, is designed for working with transcript-based genomic annotations.
     2. rtracklayer, is designed for importing and exporting annotation data into a variety of different formats.
+ 
### Retrieving Promoter Regions: Flank and Promoters
### Retrieving Promoter Sequence: Connection GenomicRanges with Sequence Data
### Getting Intergenic and Intronic Regions: Gaps, Reduce, and Setdiffs in Practice
### Finding and Working with Overlapping Ranges
### Calculating Coverage of GRanges Objects
## Working with Ranges Data on the Command Line with BEDTools
### Computing Overlaps with BEDTools Intersect
### BEDTools Slop and Flank
### Coverage with BEDTools
### Other BEDTools Subcommands and pybedtools
