### Reorder 

- Degree-aware Hybrid Graph Traversal on FPGA-HMC Platform

  - scale-free的图（如社交网络）节点度数呈指数分布，高度的点会在遍历时造成大量冗余的edge check，增加计算量和访存。

  - 针对图遍历算法，使用degree-aware adjacency list reordering减少冗余计算，degree-aware vertex index sorting用于减少访存。

    - degree-aware adjacency list reordering：邻接表重排序，按度递减排序。（启发式算法）

    ![image-20220427092857565](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220427092857565.png)

    ![image-20220420005244820](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005244820.png)
    
    - degree-aware vertex index sorting：根据顶点度将顶点放在不同的存储上，减少访存

- GraphR: Accelerating Graph Processing Using ReRAM

  - 图算法是迭代的，固有地容忍不精确性；概率计算（例如，PageRank 和协作过滤）和典型的涉及整数的图算法（例如，BFS/SSSP）都可以抵抗错误；ReRAM可以有效计算SpMV。GraphR是第一个基于ReRAM的图处理加速器。
  - （离线）通过将连续子图的边存储在一起（列优先），使得每次加载子图都只涉及顺序磁盘I/O；将同一个子图的边连续存放（COO），便于将压缩格式的边列表转换成稀疏矩阵。

  ![image-20220420005314829](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005314829.png)

- GraphSAR: A Sparsity-Aware Processing-in-Memory Architecture for Large-scale Graph Processing on ReRAMs

  - observation：节点在原始边列表中有局部性，按照节点在原始边列表中出现的顺序对节点进行排序，这样每个节点将被放在重新排序的邻接矩阵中靠近其邻居的位置

  - 与GraphR不同，不再搬运数据，以稀疏邻接矩阵的形式存储图，在存储图的ReRAM上运算；划分子图，提高存储空间的效率

  - Sparsity-aware Graph Partitioning：用边列表形式存储只有一条边的block，用block列表存储非零值多于50%的block，对不满足上述两种情况的block继续划分，直到存储整个邻接表

    ![image-20220420005439179](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005439179.png)

  - Lightweight Graph Clustering（offline reorder）：重新映射每个顶点的索引，当加载原始边列表时，将从0开始的索引分配给每个新顶点，从而减少非空子图的数量（顶点被划分为不相交的interval，边被划分为相应的block。block中的边是从一个interval的顶点到另一个interval的顶点）

  - src node和dst node同时重排

    ![image-20220420005359928](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005359928.png)

  - partition和reorder都很快，但是一次只使用crossbar的一部分，并行性受限制

- Spara: An Energy-Efficient ReRAM-Based Accelerator for Sparse Graph Analytics Applications
  - 稀疏源在于源和目标顶点索引到crossbar的字线和位线的连续映射（CWCB），而细致的reorder算法开销很大（RWRB）。本文折中，dst（列）连续映射，src随机映射

  - 通过扫描 SrcV 集中源节点的出边并将未访问的出边邻居放入 DstV 集中来重新排序矩阵的列。之后，通过类似的过程通过扫描DstV 中节点的入边邻居并将那些未访问的入边邻居放入SrcV集中来重新排序矩阵的行。

  - wordline-cut考虑活动顶点的稀疏性，快速加载活动顶点的数据

    - 采用压缩矩阵：快速访问活动顶点的信息，无需进行不必要的检查；节省空间

    ![image-20220420005541406](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005541406.png)

    ![image-20220420005554206](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005554206.png)

  - src node和dst node单独按初次访问顺序重排，更好利用局部性

- ReSpar: Reordering Algorithm for ReRAM-based Sparse Matrix-Vector Multiplication Accelerator
  - GraphSAR和Spara都旨在将节点的邻居聚合在一起，但是如何安排这些邻居之间的顺序并没有仔细设计
  - 根据行之间的相似度确定重排顺序（offline）

  ![image-20220420005650692](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005650692.png)

  ![image-20220420005705665](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005705665.png)

- GNNIE: GNN Inference Engine with Load-balancing and Graph-Specific Caching

  - 为最大限度地重复使用数据并最大限度地减少片外随机存储器访问，提出重排保证所有随机访问模式都局限于片上缓冲区，片外访存是顺序的

  ![image-20220420005742616](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005742616.png)

  - 按节点度降序排列，每次将未处理程度最高的节点放入子图，替换未处理程度最低的节点

- I-GCN: A Graph Convolutional Network Accelerator with Runtime Locality Enhancement through Islandization

  - 提出一个在线图重构算法，寻找内部连接强但是外部连接弱的节点簇
  - hub分布在L形区域，其他非零值分布在对角线区域

  ![image-20220420005901646](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005901646.png)

  ![image-20220420005913114](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005913114.png)

  ![image-20220420005934997](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005934997.png)

  ![image-20220420005953029](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005953029.png)
  
- A Closer Look at Lightweight Graph Reordering
  - 将完全基于度的排序方法称为skew-aware reordering techniques，这些方法应满足以下三点，第一点容易满足，后两点需要权衡
    - 快速
    - 高cache利用率
    - 保留图的结构
  - skew-aware reordering techniques
    - sort：按度重排所有节点
    - hub-sort：仅重排hot vertices
    - hub-cluster：仅聚集hot vertices，不排序

  ![image-20220420004620669](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420004620669.png)

  - 本文提出Degree-Based Grouping（DBG），进行粗粒度重排，既提高cache利用率，又尽量保留图的结构

  ![image-20220420004706714](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420004706714.png)

  ![image-20220420005051183](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420005051183.png)

- Accelerating Graph Processing With Lightweight Learning-Based Data Reordering
  - 之前的skew-aware reordering techniques以及DBG要么无法控制重排开销（一些数据集中hot vertices占比很大），要么没有达到最高的性能提升（DBG仍使用静态参数，不能因多个不同线程配置而变）
  - 使用random forest预测hot vertices的阈值，将hot vertices集中在同一page，不排序
  
  ![image-20220420100419414](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420100419414.png)
  
  ![image-20220420100430956](C:\Users\25814\AppData\Roaming\Typora\typora-user-images\image-20220420100430956.png)

  

​	
