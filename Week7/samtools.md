#### 1. 我们提供的bam文件COAD.ACTB.bam是单端测序分析的结果还是双端测序分析的结果？为什么？

使用 `samtools flagstat` 对 BAM 文件进行统计：

```
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# samtools flagstat COAD.ACTB.bam
185650 + 0 in total (QC-passed reads + QC-failed reads)
4923 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
185650 + 0 mapped (100.00% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

其中 paired in sequencing 表示双端测序的 reads 数量，该值为 0，说明该 BAM 文件中不存在成对 reads。该 BAM 文件为单端测序（single-end sequencing）结果。

#### 2. 查阅资料回答什么叫做"secondary alignment"？并统计提供的bam文件中，有多少条记录属于"secondary alignment?"
Secondary alignment 指的是当一条 read 可以比对到参考基因组的多个位置时，除最佳匹配外的其他比对结果。

我们上面就有写了，4923 + 0 secondary。

#### 3. 请根据hg38.ACTB.gff计算出在ACTB基因的每一条转录本中都被注释成intron的区域，以bed格式输出。并提取COAD.ACTB.bam中比对到ACTB基因intron区域的bam信息，后将bam转换为fastq文件。

```
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# awk '$3=="gene"{print $0}' hg38.ACTB.gff | gff2bed > genes.bed
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# awk '$3=="exon"{print $0}' hg38.ACTB.gff | gff2bed > exons.bed
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# bin/bedtools.static.binary subtract -a genes.bed -b exons.bed > introns.bed
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# bin/bedtools.static.binary intersect -abam COAD.ACTB.bam -b introns.bed > results.bam
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# samtools sort -n results.bam > results.sort.bam
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# samtools fastq results.sort.bam > results.fq
[M::bam2fq_mainloop] discarded 0 singletons
[M::bam2fq_mainloop] processed 15132 reads
```

#### 4. 利用COAD.ACTB.bam计算出reads在ACTB基因对应的genomic interval上的coverage，以bedgraph格式输出。

```
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# samtools sort COAD.ACTB.bam > COAD.ACTB.sort.bam
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# samtools index COAD.ACTB.sort.bam
docker-desktop:/tmp/docker-desktop-root/mnt/host/c/Users/DELL# bin/bedtools.static.binary genomecov -ibam COAD.ACTB.sort.bam -bg -split > COAD.ACTB.coverage.bedgraph
```
