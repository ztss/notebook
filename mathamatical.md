## 协方差
+ 衡量两个变量的总体误差
  $$ Cov(X,Y)=E[X-E(X)(Y-E(Y))] $$
## 协方差矩阵
+ 矩阵中每个元素是各个向量元素之间的协方差。
 $$ X=(X_1,X_2,...,X_N)^T 为n维随机变量 $$
 $$C=\begin{pmatrix}
 c_{_{11}} & c_{_{12}} & \dots & c_{_{1n}} \\
 c_{_{21}} & c_{_{22}} & \dots & c_{_{2n}} \\
 \vdots & \vdots & \ddots  & \vdots  \\
 c_{_{m1}} & c_{_{m2}} & \dots & c_{_{mn}} \\
 \end{pmatrix}$$    
 $c_{i,j}=cov(X_i,X_j)$
## 典型相关分析(CCA)
+ 如果有两个数据集x,y。维度分别为n1*m,n2*m。n1,n2为数据集的特征维度，我们可以使用两个
投影向量将两个数据集投影到一维，怎么选择投影向量呢？选择使得降维后的两个一维向量相关系数
最大的投影向量即可。
## z-scores
+ 将不同量级的数据转化为同一个量级，是一种normalization的方法。
$$ \frac{x-\mu}{\sigma} $$
