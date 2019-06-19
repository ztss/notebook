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

### Sorting Plain-Text Data with Sort
### Finding Unique Values in Uniq
### Join
### Text Processing with Awk
### Bioawk:An Awk for Biological Formats
### Stream Editing with Sed
## Advanced Shell Tricks
### Subshells
### Named Pipes and Process Substitution
## The Unix Philosophy Revisited
