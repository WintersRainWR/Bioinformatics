## 1. 为什么在 ChIP-seq 实验中需要设置 control（input）

在 ChIP-seq 实验中，免疫沉淀（IP）后获得的 DNA 序列的富集信号不一定完全来源于目标蛋白与 DNA 的特异性结合，还可能由以下因素造成：

- **基因组序列特性**：如重复序列或某些区域更容易被测序或比对  
- **测序与建库偏好**：不同区域测序深度不均（如 GC bias）  
- **抗体非特异性结合**：抗体或磁珠可能结合非目标 DNA  

因此需要设置 input 对照（未进行免疫沉淀的 DNA），用于表征背景信号。

在数据分析过程中，input 的主要作用包括：

- **消除背景噪音**（区分真实结合与假富集）  
- **校正系统偏差**（如测序深度差异）  
- **作为归一化基准**（用于计算 fold change）  

---

## 2. findPeaks 和 findMotifsGenome.pl 主要参数说明

### （1）findPeaks

- **-style**：选择寻找峰的模式，不同生物学事件产生的峰形不同  
  - `factor`：转录因子（尖峰）  
  - `histone`：组蛋白修饰（宽峰）  
  - `super`：超级增强子  

- **-i**：指定 input 对照的 tag directory，用于背景校正  

- **-o**：指定输出文件路径，也可以设置为 `auto` 自动命名  

- **-F**：fold change 阈值（IP / Input），用于筛选显著富集区域  

- **-P**：p-value 阈值，用于控制统计显著性  

- **-L**：局部 fold change 阈值，用于减少局部背景导致的假阳性  

- **-size**：指定峰的大小（bp），不同类型数据需调整  

---

### （2）findMotifsGenome.pl

- **-len**：指定搜索的 motif 长度，可以设置多个（默认 8,10,12）  

- **-size**：指定提取序列区域大小（以 peak 中心为基准，默认 200 bp）  

- **-S**：指定每种长度要寻找的 motif 数量（默认 25）  

- **-bg**：指定背景序列（如随机序列或匹配 GC 含量的序列）  

- **-p**：指定使用的线程数，加快计算速度  

---

## 总结

findPeaks 用于从 ChIP-seq 数据中识别显著富集区域（peaks），而 findMotifsGenome.pl 用于在这些区域中挖掘潜在的转录因子结合序列特征（motif）。

### （3）

```
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# makeTagDirectory /home/test/chip-seq/homework/ip /home/test/chip-seq/homework/ip.chrom_part.bam

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# makeTagDirectory /home/test/chip-seq/homework/input /home/test/chip-seq/homework/input.chrom_part.bam

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# findPeaks /home/test/chip-seq/homework/ip -style factor -o /home/test/chip-seq/output_myself/part.peak -i /home/test/chip-seq/homework/input

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# grep -v '^#' /home/test/chip-seq/output_myself/part.peak | awk 'NR>1 && $11>=8 && $12<1e-8 {print $0}'
chrIV-1 chrIV   465220  465468  +       111129.9        0.920   15510.000000    15585.0 234.1   66.57   0.00e+00        55.11   0.00e+00    0.50
chrIV-2 chrIV   1490100 1490348 +       81687.8 0.857   11468.000000    11456.0 195.1   58.72   0.00e+00        35.06   0.00e+00        0.50
chrV-1  chrV    141138  141386  +       54449.0 0.855   7647.000000     7636.0  182.3   41.88   0.00e+00        21.55   0.00e+00        0.52
chrV-2  chrV    69078   69326   +       48659.0 0.823   6837.000000     6824.0  206.5   33.05   0.00e+00        20.52   0.00e+00        0.50
chrV-3  chrV    85195   85443   +       46277.4 0.861   6493.000000     6490.0  225.6   28.77   0.00e+00        21.56   0.00e+00        0.52
chrIV-3 chrIV   1080509 1080757 +       34405.0 0.832   4830.000000     4825.0  234.1   20.61   0.00e+00        23.11   0.00e+00        0.50
chrIV-4 chrIV   599953  600201  +       26597.0 0.755   3733.000000     3730.0  190.1   19.62   0.00e+00        15.58   0.00e+00        0.50
chrV-4  chrV    321939  322187  +       24821.5 0.754   3484.000000     3481.0  177.4   19.63   0.00e+00        13.66   0.00e+00        0.50
chrIV-5 chrIV   1468786 1469034 +       23595.0 0.794   3317.000000     3309.0  193.7   17.08   0.00e+00        14.36   0.00e+00        0.51
chrIV-6 chrIV   132817  133065  +       19402.3 0.782   2723.000000     2721.0  209.3   13.00   0.00e+00        11.95   0.00e+00        0.52
chrIV-7 chrIV   591669  591917  +       18304.2 0.792   2568.000000     2567.0  200.1   12.83   0.00e+00        12.88   0.00e+00        0.50
chrIV-8 chrIV   721812  722060  +       17840.7 0.739   2514.000000     2502.0  192.3   13.01   0.00e+00        9.61    0.00e+00        0.51
chrIV-9 chrIV   1       230     +       17206.1 0.884   2444.000000     2430.0  60.3    40.30   0.00e+00        632.81  0.00e+00        0.81
chrIV-10        chrIV   1233763 1234011 +       16250.6 0.888   2303.000000     2279.0  156.1   14.60   0.00e+00        10.68   0.00e+00    0.52
chrIV-11        chrIV   234340  234588  +       16179.3 0.790   2275.000000     2269.0  199.4   11.38   0.00e+00        9.37    0.00e+00    0.51
chrV-5  chrV    225453  225701  +       15066.9 0.901   2123.000000     2113.0  157.5   13.42   0.00e+00        11.73   0.00e+00        0.53
chrIV-12        chrIV   357166  357414  +       15052.6 0.705   2113.000000     2111.0  205.7   10.26   0.00e+00        8.19    0.00e+00    0.51
chrIV-13        chrIV   416932  417180  +       13954.5 0.838   1973.000000     1957.0  188.0   10.41   0.00e+00        10.83   0.00e+00    0.51
chrIV-15        chrIV   1278678 1278926 +       13697.8 0.852   1925.000000     1921.0  225.6   8.51    0.00e+00        12.86   0.00e+00    0.52
chrIV-16        chrIV   1164971 1165219 +       13419.7 0.746   1887.000000     1882.0  215.7   8.73    0.00e+00        9.29    0.00e+00    0.52
chrV-6  chrV    491091  491339  +       12457.1 0.773   1749.000000     1747.0  180.9   9.66    0.00e+00        13.69   0.00e+00        0.52
chrIV-14        chrIV   1525285 1525496 +       11779.7 0.923   1953.000000     1652.0  58.2    28.40   0.00e+00        32.51   0.00e+00    1.27
chrIV-17        chrIV   722439  722687  +       10774.3 0.676   1512.000000     1511.0  172.4   8.76    0.00e+00        5.26    0.00e+00    0.53
chrIV-46        chrIV   568825  569073  +       5005.7  0.820   705.000000      702.0   85.8    8.18    0.00e+00        4.54    3.86e-226   0.84

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# findMotifsGenome.pl /home/test/chip-seq/output_myself/part.peak sacCer2 /home/test/chip-seq/output_myself/part.motif.output -len 8
```
