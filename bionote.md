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
