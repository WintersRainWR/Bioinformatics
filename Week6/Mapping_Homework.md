### 一、Bowtie 中 BWT 的作用及内存优化

Bowtie 使用 Burrows–Wheeler Transform（BWT）以及 FM-index 来提高比对效率。BWT 的一个重要特点是可以支持从后向前的字符串匹配（backward search）。在比对过程中，每加入一个碱基，搜索区间都会迅速缩小，从而避免对整个基因组进行逐位扫描。这使得比对时间主要取决于 read 的长度，而不是参考基因组的大小。

此外，BWT 会将具有相同前缀的序列聚集在一起，从而进一步减少搜索空间，提高查找效率。

在内存优化方面，Bowtie 主要采用了以下策略：

使用 FM-index 对基因组进行压缩存储，而不是直接存储完整序列。
采用稀疏的 suffix array（后缀数组采样），只保存部分位置信息，其余通过计算恢复。
使用 2-bit 编码表示 DNA 碱基（A/C/G/T），降低存储空间。
对索引进行结构优化，使其能够在较小内存下运行。

这些方法使得 Bowtie 在保持较高速度的同时，显著降低了内存占用。

### 二、用 Bowtie 将 THA2.fa mapping 到 BowtieIndex/YeastGenome 上，得到 THA2.sam，统计mapping到不同染色体上的reads数量(即统计每条染色体都map上了多少条reads)。

```
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# bowtie -v 2 -m 10 --best --strata BowtieIndex/YeastGenome -f THA2.fa -S THA2.sam
# reads processed: 1250
# reads with at least one reported alignment: 1158 (92.64%)
# reads that failed to align: 77 (6.16%)
# reads with alignments suppressed due to -m: 15 (1.20%)
Reported 1158 alignments to 1 output stream(s)

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# awk 'NR>19 && $3!="*"{print $3}' THA2.sam | sort | uniq -c
     18 chrI
     51 chrII
     15 chrIII
    194 chrIV
     25 chrIX
     12 chrmt
     33 chrV
     17 chrVI
    125 chrVII
     68 chrVIII
     71 chrX
     56 chrXI
    169 chrXII
     67 chrXIII
     58 chrXIV
    101 chrXV
     78 chrXVI
```

### 一、相关概念说明

#### 3.1 CIGAR string

CIGAR（Compact Idiosyncratic Gapped Alignment Report）字符串用于描述 read 与参考基因组之间的比对方式，记录了匹配、插入、缺失等信息。

常见操作包括：

- M：匹配或错配  
- I：插入（相对于参考序列）  
- D：缺失  
- S：soft clipping  
- H：hard clipping  

例如：


50M


表示 50 个碱基全部参与匹配；


45M5S


表示前 45 个碱基参与比对，后 5 个碱基被裁剪。

---

#### 3.2 Soft clip

Soft clip 指的是 read 的一部分没有参与比对，但仍然保留在序列中。

在 CIGAR 字符串中使用字母 `S` 表示，例如：


5S45M


表示前 5 个碱基未参与比对，后 45 个碱基成功比对。

与之对应，hard clip（`H`）表示被裁剪的序列不会保留在结果中。

---

#### 3.3 Mapping Quality

Mapping Quality（MAPQ）用于表示比对结果的可靠性，其定义为：

$$
MAPQ = -10 \\log_{10}(P)
$$

其中 \( P \) 表示比对错误的概率。

MAPQ 值越高，说明该 read 被正确比对到该位置的可能性越大。一般来说：

- MAPQ 较高：通常为唯一比对，可信度较高  
- MAPQ 较低：可能存在多个候选比对位置  

---

#### 3.4 是否可以从 SAM 文件恢复参考序列

仅根据 SAM/BAM 文件，无法完整恢复参考基因组序列。

虽然 SAM 文件中包含 read 序列、比对位置（POS）以及 CIGAR 信息，可以反映 read 与参考序列之间的差异，但由于缺少参考基因组的完整序列信息，因此无法还原完整的参考序列。

### 4. 软件安装和资源文件的下载也是生物信息学实践中的重要步骤。请自行安装教程中未涉及的bwa软件，从UCSC Genome Browser下载Yeast (S. cerevisiae, sacCer3)基因组序列。使用bwa对Yeast基因组sacCer3.fa建立索引，并利用bwa将THA2.fa，mapping到Yeast参考基因组上，并进一步转化输出得到THA2-bwa.sam文件。
```
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# bwa/bwa-0.7.19/bwa index sacCer3.fa
[bwa_index] Pack FASTA... 0.04 sec
[bwa_index] Construct BWT for the packed sequence...
[bwa_index] 3.55 seconds elapse.
[bwa_index] Update BWT... 0.03 sec
[bwa_index] Pack forward-only FASTA... 0.02 sec
[bwa_index] Construct SA from BWT and Occ... 2.02 sec
[main] Version: 0.7.19-r1273
[main] CMD: bwa/bwa-0.7.19/bwa index sacCer3.fa
[main] Real time: 5.727 sec; CPU: 5.678 sec

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# bwa/bwa-0.7.19/bwa aln -t 24 sacCer3.fa THA2.fa > THA2-bwa.sai
[bwa_aln] 17bp reads: max_diff = 2
[bwa_aln] 38bp reads: max_diff = 3
[bwa_aln] 64bp reads: max_diff = 4
[bwa_aln] 93bp reads: max_diff = 5
[bwa_aln] 124bp reads: max_diff = 6
[bwa_aln] 157bp reads: max_diff = 7
[bwa_aln] 190bp reads: max_diff = 8
[bwa_aln] 225bp reads: max_diff = 9
[bwa_aln_core] calculate SA coordinate... 0.17 sec
[bwa_aln_core] write to the disk... 0.00 sec
[bwa_aln_core] 1250 sequences have been processed.
[main] Version: 0.7.19-r1273
[main] CMD: bwa/bwa-0.7.19/bwa aln -t 24 sacCer3.fa THA2.fa
[main] Real time: 0.046 sec; CPU: 0.177 sec

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# bwa/bwa-0.7.19/bwa samse sacCer3.fa THA2-bwa.sai THA2.fa > THA2-bwa.sam
[bwa_aln_core] convert to sequence coordinate... 0.02 sec
[bwa_aln_core] refine gapped alignments... 0.00 sec
[bwa_aln_core] print alignments... 0.00 sec
[bwa_aln_core] 1250 sequences have been processed.
[main] Version: 0.7.19-r1273
[main] CMD: bwa/bwa-0.7.19/bwa samse sacCer3.fa THA2-bwa.sai THA2.fa
[main] Real time: 0.024 sec; CPU: 0.023 sec

docker-desktop:/tmp/docker-desktop-root/mnt/host/c/WINDOWS/system32# tail THA2-bwa.sam
2491-297        16      chrXII  795497  37      24M     *       0       0       CTATTTAGTTATTATATGATTAAA        *       XT:A:U  NM:i:1  X0:i:1  X1:i:0    XM:i:1  XO:i:0  XG:i:0  MD:Z:7A16
2492-296        0       chrXIV  283182  37      31M     *       0       0       TTCTTTTTCTCTTTTCTTTATACAAAAAATG *       XT:A:U  NM:i:1  X0:i:1  X1:i:0    XM:i:1  XO:i:0  XG:i:0  MD:Z:15T15
2493-296        0       chrII   76544   0       17M     *       0       0       AAAAAAACAACAATTTG       *       XT:A:R  NM:i:1  X0:i:2  X1:i:43 XM:i:1    XO:i:0  XG:i:0  MD:Z:2G14
2494-296        16      chrII   591669  37      27M     *       0       0       CTAAAAGGAAAAAGTGTAAAAATTATG     *       XT:A:U  NM:i:1  X0:i:1  X1:i:0    XM:i:1  XO:i:0  XG:i:0  MD:Z:15A11
2495-296        16      chrXII  369647  37      27M     *       0       0       CAAAAAAGTTTGTAAAAAATTTAGAAT     *       XT:A:U  NM:i:1  X0:i:1  X1:i:0    XM:i:1  XO:i:0  XG:i:0  MD:Z:11A15
2496-296        0       chrXII  1029961 37      30M     *       0       0       TATCTATAATTTTTTTTTTCTCTTACATAG  *       XT:A:U  NM:i:0  X0:i:1  X1:i:0    XM:i:0  XO:i:0  XG:i:0  MD:Z:30
2497-296        0       chrXIV  577168  37      22M     *       0       0       TACCATATATATTAAAAATAAG  *       XT:A:U  NM:i:1  X0:i:1  X1:i:0  XM:i:1    XO:i:0  XG:i:0  MD:Z:3T18
2498-296        16      chrVII  1048720 37      29M     *       0       0       CTTAATTTAAAGTTAAAAATGTTGGGTAT   *       XT:A:U  NM:i:1  X0:i:1  X1:i:0    XM:i:1  XO:i:0  XG:i:0  MD:Z:20A8
2499-296        16      chrIV   720943  37      22M     *       0       0       CTAGAATTAGAGTAAGAAATTA  *       XT:A:U  NM:i:1  X0:i:1  X1:i:0  XM:i:1    XO:i:0  XG:i:0  MD:Z:11A10
2500-296        16      chrV    67079   37      7M2D28M *       0       0       CATATAAATATATATATATTTGTATAAAAGGTGTA     *       XT:A:U  NM:i:2  X0:i:1    X1:i:0  XM:i:0  XO:i:1  XG:i:2  MD:Z:7^AT28
```
