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
  colnames(d)[12] <- "percent.GC"
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

### Exploring Data Visually with ggplot2 II: Smoothing
### Binning Data with cut() and Bar Plots with ggplot2
### Merging and Combining Data: Matching Vectors and Merging Dataframes
### Using ggplot2 Facets
### More R Data Structures: Lists
### Writing and Applying Functions to Lists with lapply() and sapply()
#### Using lapply()
#### Writing functions
#### Digression: Debugging R Code
#### More list apply functions: sapply() and mapply()
### Working with the Split-Apply-Combine Pattern
### Exploring Dataframes with dplyr
### Working with Strings
## Developing Workflows with R Scripts
### Control Flow: if, for, and while
### Working with R Scripts
### Workflows for Loading and Combining Multiple Files
### Exporting Data
## Further R Directions and Resources
