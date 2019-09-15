## A non-negative matrix factorization method for detecting modules in heterogeneous omics multi-modal data
+ As a result, NMF solutions are less uniquely defined but are more interpretable.
+ given non-negative data matrix X N×M, NMF finds a non-negative factorization WH
of rank D that best approximates X.
+ Frobenius Norm:  $\parallel X \parallel_{F}\stackrel{def}{=}\sqrt{\sum_{i}\sum_{j}x_{i,j}}$
可以用于利用低秩矩阵来近似单一矩阵，即用一个秩为k的矩阵B,使得矩阵B与原始矩阵的差的范数尽可能
小。
$$B=arg \space min_{rank(B)=k}\parallel{A-B}\parallel_{F}$$
+ The parameter λ can be viewed as the homogeneity parameter, since larger values
induce smaller VkHk⁠. When datasets from multiple sources contain homogeneous elements,
performing separate analyses (λ = 0) sacrifices power; when datasets contain
heterogeneous elements, a purely joint analysis (⁠λ=+∞⁠) is sensitive to extraneous noise.
+ Since the iNMF objective function is non-convex, one should perform many
repetitions and choose the minimizer of the objective function as the final solution.
