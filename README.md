## 1.基因组去冗余 dRep
>https://github.com/MrOlm/drep

上接大规模组装的MAG结果，获得物种水平的代表基因组。



为避免第二步ANI聚类中cluster过大导致ANI计算过慢，采用两步聚类的方法：

首先按mash 0.95；ANImf 0.99聚类获得strain(strain定义存在争议)水平的结果；

再将strain水平的代表基因组按mash 0.90；ANImf 0.95聚类获得species的代表基因组。



MAG总数超过两万后，可按两万个MAG一组分别聚类，最后再将所有组合并。若基因组收集自多个数据库，数据库之间存在冗余的话，可以使用mash以0.999的相似度先去一遍冗余的基因组。

两万个MAG一组，每组的执行时间约为10天。可在log文件中估算计算时间。

```
# conda install -c bioconda dRep #v3.2.0

cd 1.dRep
mkdir 1.MAGs 2.dRep_strain 3.dRep_species
python format2dRep.py ../0.data/> MAGs_dRep.csv

# 0.99 for strain


# 0.95 for species

```
最新的dRep v3.2.0没有提供分步计算的脚本了。

下列是挑选分离单菌的脚本

```
dRep dereplicate \
dRep \
--genomes genomes/*.fna \
--processors 6 \
--completeness 50 \
--contamination 10 \
--S_algorithm ANImf \
--SkipMash \
--S_ani 0.9 \
--cov_thresh 0.3 \
--completeness_weight 1 \
--contamination_weight 5 \
--strain_heterogeneity_weight 0 \
--N50_weight 0.5 \
--size_weight 0.5 \
--centrality_weight 0.5
```



## 2. 分类学注释

### 2.1 GTDBtk
> https://github.com/Ecogenomics/GTDBTk
> GTDB-Tk v1.5.0 was released on April 23, 2021 along with new reference data for GTDB R06-RS202.
> Please note v1.5.0+ is not compatible with GTDB R05-RS95.



## 3.基因组注释
prokka

## 4.pangenome

## 5. 基因组比较

### 5.1 Mash

```
mash sketch -s 1e5 -k 21 -p 8 -o ../5.mash/B.uniformis.k21.s1e5 *.fa
mash
```

### 5.2 fastANI



## 6.构建进化树
### 3.1 PhyloPhlAn
版本: phylophlan-3.0.2
加载环境变量。
```
export PATH="/ldfssz1/ST_META/share/User/tianliu/toolkit/phylophlan-3.0.2/phylophlan:$PATH"
```

phylophlan的数据库下载链接：https://forum.biobakery.org/t/pre-downloading-phylophlan-databases/1587

#### 3.1.1 构建门级别的进化树
构建原核生物(细菌+古菌)的进化树，每个节点为一个物种的代表基因组(SGB)。

#### 3.1.2 构建单个物种内部的进化树
高分辨率，每个节点为一个基因组(菌株/MAGs)。
1. 使用prodigal预测基因
```
prodigal -c -m -p single -f gff -i 1.genome/GUT_GENOME103119.fa -a GUT_GENOME103119.faa > GUT_GENOME103119.gff
```

2. 使用该物种的核心基因集作为比对数据库。可直接从该物种的pangenome中获得。
```
cd /ldfssz1/ST_META/share/User/tianliu/bioenv/conda_centos7/envs/bioenv/lib/python3.8/site-packages/phylophlan/phylophlan_databases
mkdir Clostridium_Q

```

3. 生成config文件。
```
phylophlan_write_config_file \
    -o references_config.cfg \
    -d a \
    --db_aa diamond \
    --map_aa diamond \
    --map_dna diamond \
    --msa mafft \
    --trim trimal \
    --tree fasttree
```
4. 构建进化树。
```
nohup phylophlan.py -i 2.gene -o 4.phyophlan -d phylophlan -t a -f references_config.cfg --nproc 16 --min_num_markers 80 --fast --diversity low --verbose &
```

### 
