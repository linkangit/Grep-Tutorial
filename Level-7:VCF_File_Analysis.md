# Level 7: VCF File Analysis

VCF (Variant Call Format) files contain genetic variation data and are crucial for population genomics, medical genetics, and evolutionary studies. This level focuses on using grep to analyze variants, genotypes, and population-level genomic data.

## Learning Objectives

By the end of this level, you will be able to:
- Parse VCF file structure and extract variant information
- Filter variants by quality metrics and genomic coordinates
- Analyze genotype patterns and allele frequencies
- Extract population-specific variant data
- Perform quality control on variant datasets
- Identify clinically relevant variants

## VCF File Structure

### VCF Format Overview
VCF files consist of:
1. **Header lines** (starting with ##)
2. **Column header line** (starting with #)
3. **Data lines** (variant records)

### Example VCF Structure:
```
##fileformat=VCFv4.2
##reference=GRCh38
##INFO=<ID=AF,Number=A,Type=Float,Description="Allele Frequency">
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	Sample1	Sample2	Sample3
chr1	1000	rs123	A	G	99.9	PASS	AF=0.25	GT:DP:GQ	0/1:30:99	0/0:25:95	1/1:20:90
chr1	2000	.	T	C	85.5	PASS	AF=0.10	GT:DP:GQ	0/0:35:99	0/1:28:88	0/0:30:92
```

### VCF Columns (1-9 are mandatory):
1. **CHROM** - Chromosome
2. **POS** - Position (1-based)
3. **ID** - Variant identifier (rs number or .)
4. **REF** - Reference allele
5. **ALT** - Alternative allele(s)
6. **QUAL** - Quality score
7. **FILTER** - Filter status (PASS/FAIL)
8. **INFO** - Additional information
9. **FORMAT** - Format for sample data
10+ **Sample columns** - Genotype data for each sample

## Basic VCF Parsing

### 1. Header and Metadata Analysis

#### Extract Header Information
```bash
# Show all header lines
grep "^##" variants.vcf

# Extract specific header types
grep "^##fileformat" variants.vcf
grep "^##reference" variants.vcf
grep "^##contig" variants.vcf

# Get INFO field definitions
grep "^##INFO" variants.vcf

# Extract FORMAT field definitions
grep "^##FORMAT" variants.vcf

# Get sample names from column header
grep "^#CHROM" variants.vcf | cut -f10-
```

#### Count Samples and Variants
```bash
# Count total variants (excluding headers)
grep -v "^#" variants.vcf | wc -l

# Count samples
grep "^#CHROM" variants.vcf | awk '{print NF-9}'

# Show column structure
grep "^#CHROM" variants.vcf | tr '\t' '\n' | nl
```

### 2. Basic Variant Filtering

#### Chromosome and Position Filtering
```bash
# Extract variants from specific chromosome
grep -v "^#" variants.vcf | grep "^chr1"

# Multiple chromosomes
grep -v "^#" variants.vcf | grep -E "^(chr1|chr2|chr3)"

# Autosomal chromosomes only
grep -v "^#" variants.vcf | grep -E "^chr([1-9]|1[0-9]|2[0-2])"

# Sex chromosomes
grep -v "^#" variants.vcf | grep -E "^(chrX|chrY)"

# Specific genomic region (chr1:1000000-2000000)
grep -v "^#" variants.vcf | awk '$1=="chr1" && $2>=1000000 && $2<=2000000'
```

#### Variant Type Analysis
```bash
# SNPs (single nucleotide variants)
grep -v "^#" variants.vcf | awk 'length($4)==1 && length($5)==1'

# Insertions (REF shorter than ALT)
grep -v "^#" variants.vcf | awk 'length($4) < length($5)'

# Deletions (REF longer than ALT)
grep -v "^#" variants.vcf | awk 'length($4) > length($5)'

# Complex variants (multiple nucleotides)
grep -v "^#" variants.vcf | awk 'length($4)>1 && length($5)>1 && length($4)==length($5)'
```

### 3. Quality Filtering

#### Basic Quality Metrics
```bash
# High quality variants (QUAL > 30)
grep -v "^#" variants.vcf | awk '$6 > 30'

# Very high quality variants (QUAL > 99)
grep -v "^#" variants.vcf | awk '$6 > 99'

# Filter by FILTER column (only PASS variants)
grep -v "^#" variants.vcf | grep -E "\tPASS\t"

# Exclude filtered variants
grep -v "^#" variants.vcf | grep -v -E "\t(LowQual|StrTest|SB)\t"
```

#### Advanced Quality Filtering
```bash
# Variants with specific depth (DP > 20)
grep -v "^#" variants.vcf | grep "DP=[2-9][0-9]"

# High mapping quality (MQ > 50)
grep -v "^#" variants.vcf | grep -E "MQ=[5-9][0-9]"

# Variants with sufficient allele balance
grep -v "^#" variants.vcf | grep -E "AB=0\.[3-7]"
```

## Advanced VCF Analysis

### 4. INFO Field Parsing

#### Allele Frequency Analysis
```bash
# Extract allele frequencies
grep -v "^#" variants.vcf | grep -oE "AF=[0-9]*\.?[0-9]+"

# High frequency variants (AF > 0.05)
grep -v "^#" variants.vcf | grep -E "AF=0\.[1-9]|AF=[1-9]"

# Rare variants (AF < 0.01)
grep -v "^#" variants.vcf | grep -E "AF=0\.00[0-9]"

# Common variants (AF > 0.05)
grep -v "^#" variants.vcf | awk '/AF=/ {
    match($8, /AF=([0-9]*\.?[0-9]+)/, arr); 
    if(arr[1] > 0.05) print
}'
```

#### Functional Impact Analysis
```bash
# Extract consequence annotations (if present)
grep -v "^#" variants.vcf | grep -oE "CSQ=[^;]*"

# Find missense variants
grep -v "^#" variants.vcf | grep "missense"

# Find loss-of-function variants
grep -v "^#" variants.vcf | grep -E "(stop_gained|frameshift|splice)"

# Find synonymous variants
grep -v "^#" variants.vcf | grep "synonymous"
```

#### Population Genetics Metrics
```bash
# Hardy-Weinberg equilibrium p-values
grep -v "^#" variants.vcf | grep -oE "HWE=[0-9e.-]+"

# Inbreeding coefficient
grep -v "^#" variants.vcf | grep -oE "InbreedingCoeff=[0-9.-]+"

# Number of alleles
grep -v "^#" variants.vcf | grep -oE "AN=[0-9]+"

# Allele count
grep -v "^#" variants.vcf | grep -oE "AC=[0-9]+"
```

### 5. Genotype Analysis

#### Basic Genotype Patterns
```bash
# Homozygous reference (0/0)
grep -v "^#" variants.vcf | grep "0/0"

# Heterozygous (0/1 or 1/0)
grep -v "^#" variants.vcf | grep -E "(0/1|1/0)"

# Homozygous alternate (1/1)
grep -v "^#" variants.vcf | grep "1/1"

# Missing genotypes (./.)
grep -v "^#" variants.vcf | grep "\./\."
```

#### Sample-Specific Analysis
```bash
# Extract genotypes for specific sample (column 10 = first sample)
grep -v "^#" variants.vcf | cut -f1-9,10

# Find variants where specific sample is heterozygous
grep -v "^#" variants.vcf | awk '$10 ~ /0\/1/ || $10 ~ /1\/0/'

# Count variants per sample
for i in {10..15}; do
    echo "Sample $((i-9)): $(grep -v "^#" variants.vcf | awk -v col=$i '$col !~ /0\/0/ && $col !~ /\.\/\./' | wc -l)"
done
```

#### Multi-Sample Genotype Analysis
```bash
# Find variants where all samples are heterozygous
grep -v "^#" variants.vcf | awk '{
    het=1; 
    for(i=10; i<=NF; i++) {
        if($i !~ /0\/1/ && $i !~ /1\/0/) het=0
    } 
    if(het) print
}'

# Find private variants (only one sample has alternate allele)
grep -v "^#" variants.vcf | awk '{
    count=0; 
    for(i=10; i<=NF; i++) {
        if($i ~ /1/) count++
    } 
    if(count==1) print
}'

# Calculate minor allele frequency across samples
grep -v "^#" variants.vcf | awk '{
    alt=0; total=0;
    for(i=10; i<=NF; i++) {
        if($i ~ /[01]\/[01]/) {
            split($i, gt, /[\/|]/);
            total += 2;
            if(gt[1] == "1") alt++;
            if(gt[2] == "1") alt++;
        }
    }
    if(total > 0) maf = alt/total;
    if(maf > 0.5) maf = 1-maf;
    print $1, $2, $4, $5, maf
}'
```

## Clinical and Medical Genetics Applications

### 6. Disease Variant Analysis

#### Known Disease Variants
```bash
# Find variants with ClinVar annotations
grep -v "^#" variants.vcf | grep -i "clinvar"

# Pathogenic variants
grep -v "^#" variants.vcf | grep -i "pathogenic"

# Benign variants
grep -v "^#" variants.vcf | grep -i "benign"

# Variants of uncertain significance
grep -v "^#" variants.vcf | grep -i "uncertain"

# Extract dbSNP IDs for known variants
grep -v "^#" variants.vcf | grep -oE "rs[0-9]+"
```

#### Cancer-Related Variants
```bash
# Find variants in cancer genes (example gene list)
cancer_genes="TP53|BRCA1|BRCA2|EGFR|KRAS|PIK3CA"
grep -v "^#" variants.vcf | grep -iE "gene=(${cancer_genes})"

# Somatic vs germline classification
grep -v "^#" variants.vcf | grep "SOMATIC"
grep -v "^#" variants.vcf | grep -v "SOMATIC"  # Germline by exclusion
```

#### Pharmacogenomics
```bash
# Find variants affecting drug metabolism
grep -v "^#" variants.vcf | grep -iE "(CYP|UGT|DPYD|TPMT)"

# Extract pharmacogenomic annotations
grep -v "^#" variants.vcf | grep -oE "PharmGKB=[^;]*"
```

### 7. Population Genetics Analysis

#### Ancestry and Population Structure
```bash
# Extract population-specific allele frequencies
grep -v "^#" variants.vcf | grep -oE "(EUR_AF|AFR_AF|ASN_AF|AMR_AF)=[0-9]*\.?[0-9]+"

# Find population-specific variants
grep -v "^#" variants.vcf | grep "EUR_AF=0" | grep -E "(AFR_AF=[1-9]|ASN_AF=[1-9])"

# Identify admixture informative markers
grep -v "^#" variants.vcf | awk '/EUR_AF=/ && /AFR_AF=/ {
    match($8, /EUR_AF=([0-9]*\.?[0-9]+)/, eur);
    match($8, /AFR_AF=([0-9]*\.?[0-9]+)/, afr);
    if(abs(eur[1] - afr[1]) > 0.3) print
}'
```

#### Selection and Evolution
```bash
# Find variants under selection (high Fst)
grep -v "^#" variants.vcf | grep -E "Fst=[0-9]*\.[5-9]"

# Ancient DNA variants
grep -v "^#" variants.vcf | grep -i "ancient"

# Balancing selection signatures
grep -v "^#" variants.vcf | grep -E "Tajima_D=[2-9]"
```

## Structural Variants and Complex Events

### 8. Large Variant Analysis

#### Structural Variant Types
```bash
# Deletions longer than 50bp
grep -v "^#" variants.vcf | grep -E "SVTYPE=DEL" | awk '$8 ~ /SVLEN=-[5-9][0-9]/ || $8 ~ /SVLEN=-[0-9]{3,}/'

# Insertions
grep -v "^#" variants.vcf | grep "SVTYPE=INS"

# Duplications
grep -v "^#" variants.vcf | grep "SVTYPE=DUP"

# Inversions
grep -v "^#" variants.vcf | grep "SVTYPE=INV"

# Translocations
grep -v "^#" variants.vcf | grep "SVTYPE=TRA"
```

#### Copy Number Variants
```bash
# Extract copy number information
grep -v "^#" variants.vcf | grep -oE "CN=[0-9]+"

# Find copy number losses (CN < 2)
grep -v "^#" variants.vcf | grep -E "CN=[01]"

# Find copy number gains (CN > 2)
grep -v "^#" variants.vcf | grep -E "CN=[3-9]"
```

## Quality Control and Validation

### 9. VCF Quality Assessment

#### Missing Data Analysis
```bash
# Count missing genotypes per variant
grep -v "^#" variants.vcf | awk '{
    missing=0;
    for(i=10; i<=NF; i++) {
        if($i ~ /\.\/\./) missing++
    }
    print $1, $2, missing, NF-9, missing/(NF-9)
}'

# Variants with high missing rate (>10%)
grep -v "^#" variants.vcf | awk '{
    missing=0; total=NF-9;
    for(i=10; i<=NF; i++) {
        if($i ~ /\.\/\./) missing++
    }
    if(missing/total > 0.1) print
}'
```

#### Mendelian Error Detection
```bash
# Simple trio analysis (parents: samples 1,2; child: sample 3)
grep -v "^#" variants.vcf | awk '{
    father=$10; mother=$11; child=$12;
    # Check for basic Mendelian violations
    if(father ~ /0\/0/ && mother ~ /0\/0/ && child ~ /1/) {
        print "Mendelian error at " $1 ":" $2
    }
}'
```

#### Batch Effect Detection
```bash
# Compare variant counts between sample groups
# Assuming samples 1-5 are batch1, 6-10 are batch2
grep -v "^#" variants.vcf | awk '{
    batch1_vars=0; batch2_vars=0;
    for(i=10; i<=14; i++) if($i !~ /0\/0/) batch1_vars++;
    for(i=15; i<=19; i++) if($i !~ /0\/0/) batch2_vars++;
    print $1, $2, batch1_vars, batch2_vars
}'
```

## Practice Exercises

### Exercise 1: Basic VCF Analysis
Using `sample_variants.vcf`:
1. Count total number of variants
2. Extract all SNPs on chromosome 1
3. Find high-quality variants (QUAL > 50)
4. Count variants that passed all filters

**Sample Solutions:**
```bash
# 1. Count variants
grep -v "^#" sample_variants.vcf | wc -l

# 2. SNPs on chr1
grep -v "^#" sample_variants.vcf | grep "^chr1" | awk 'length($4)==1 && length($5)==1'

# 3. High quality variants
grep -v "^#" sample_variants.vcf | awk '$6 > 50'

# 4. PASS variants
grep -v "^#" sample_variants.vcf | grep -E "\tPASS\t" | wc -l
```

### Exercise 2: Genotype Analysis
Using `population_variants.vcf`:
1. Find all heterozygous variants in sample 1
2. Calculate the number of private variants per sample
3. Identify variants where all samples are homozygous alternate
4. Find variants with missing data in >50% of samples

### Exercise 3: Clinical Variant Analysis
Using `clinical_variants.vcf`:
1. Extract all pathogenic variants
2. Find variants in known cancer susceptibility genes
3. Identify compound heterozygous variants in the same gene
4. Extract pharmacogenomic variants affecting drug response

## Real-World Applications

### Genome-Wide Association Studies (GWAS)
```bash
# Extract variants for association testing
grep -v "^#" gwas_variants.vcf | awk '$6 > 30 && /PASS/' > high_quality_variants.vcf

# Find variants with MAF in appropriate range for GWAS (0.01 < MAF < 0.5)
grep -v "^#" variants.vcf | awk '/AF=/ {
    match($8, /AF=([0-9]*\.?[0-9]+)/, arr);
    maf = arr[1];
    if(maf > 0.5) maf = 1 - maf;
    if(maf > 0.01 && maf < 0.5) print
}'
```

### Rare Disease Analysis
```bash
# Find rare variants (MAF < 0.001) in coding regions
grep -v "^#" exome_variants.vcf | awk '/AF=/ {
    match($8, /AF=([0-9]*\.?[0-9]+)/, arr);
    if(arr[1] < 0.001) print
}'

# Compound heterozygous analysis
# Extract variants where patient is heterozygous in disease genes
grep -v "^#" patient_variants.vcf | awk '$10 ~ /0\/1/ || $10 ~ /1\/0/' | grep -E "gene=(CFTR|BRCA1|BRCA2)"
```

### Pharmacogenomics
```bash
# Extract star allele variants
grep -v "^#" pgx_variants.vcf | grep -E "\*[0-9]+"

# Find variants affecting warfarin dosing
grep -v "^#" variants.vcf | grep -E "(CYP2C9|VKORC1|CYP4F2)"

# Identify poor metabolizer genotypes
grep -v "^#" variants.vcf | grep "CYP2D6" | awk '$10 ~ /1\/1/ || $11 ~ /1\/1/'
```

## Advanced VCF Processing

### Multi-VCF Analysis
```bash
# Compare variants between two VCF files
# Extract positions from each file and compare
grep -v "^#" sample1.vcf | cut -f1,2 | sort > sample1_positions.txt
grep -v "^#" sample2.vcf | cut -f1,2 | sort > sample2_positions.txt
comm -12 sample1_positions.txt sample2_positions.txt  # Shared variants
comm -23 sample1_positions.txt sample2_positions.txt  # Sample1 only
```

### Complex Inheritance Patterns
```bash
# X-linked recessive analysis (males should be hemizygous)
grep -v "^#" variants.vcf | grep "^chrX" | awk '{
    # Assuming male samples are known (samples 1,3,5)
    for(i in [10,12,14]) {
        if($i ~ /0\/1/) print "Heterozygous male at " $1 ":" $2
    }
}'

# Autosomal recessive compound heterozygous
# Find genes with multiple heterozygous variants in same individual
grep -v "^#" variants.vcf | awk '$10 ~ /0\/1/ || $10 ~ /1\/0/' | cut -f1,2,8 | grep -oE "gene=[^;]*" | sort | uniq -c | awk '$1 > 1'
```

## Performance Optimization for Large VCFs

```bash
# Index-based processing for large files
# Use tabix for rapid region extraction (requires preprocessing)
# grep -v "^#" variants.vcf | bgzip > variants.vcf.gz
# tabix -p vcf variants.vcf.gz

# Stream processing for memory efficiency
# Process one chromosome at a time for large files
for chr in {1..22} X Y; do
    grep -v "^#" huge_variants.vcf | grep "^chr${chr}" | process_chromosome.sh
done

# Parallel processing by chromosome
parallel -j 8 'grep -v "^#" variants.vcf | grep "^chr{}" > chr{}_variants.vcf' ::: {1..22} X Y
```

## Quick Reference - VCF Patterns

| Task | Pattern | Example |
|------|---------|---------|
| Extract variants | `grep -v "^#"` | Skip header lines |
| Chromosome filter | `grep "^chr1"` | Chr1 variants only |
| SNPs only | `awk 'length($4)==1 && length($5)==1'` | Single nucleotide variants |
| Quality filter | `awk '$6 > 30'` | QUAL > 30 |
| PASS variants | `grep "\tPASS\t"` | Passed filters |
| Allele frequency | `grep -oE "AF=[0-9.]*"` | Extract AF values |
| Heterozygous | `grep -E "(0/1\|1/0)"` | Het genotypes |
| Missing genotypes | `grep "\./\."` | Missing data |
| dbSNP IDs | `grep -oE "rs[0-9]+"` | Extract rs numbers |

## Tips for VCF Processing

1. **Always skip headers**: Use `grep -v "^#"` to exclude header lines
2. **Handle large files**: Consider splitting by chromosome for processing
3. **Validate coordinates**: Check for valid genomic positions
4. **Genotype formats**: Understand phased (|) vs unphased (/) genotypes
5. **Missing data**: Account for ./. and partial missing genotypes
6. **Multi-allelic sites**: Handle complex ALT field structures
7. **Tool integration**: Combine with bcftools, vcftools for complex operations

## Next Steps

You've now mastered VCF analysis with grep! You're ready for Level 8, where we'll focus on performance optimization techniques for processing large genomic datasets efficiently, including memory management and speed optimization strategies.

## Summary of Level 7 Skills

- ✅ Parse VCF file structure and metadata
- ✅ Filter variants by quality, coordinates, and type
- ✅ Analyze genotype patterns across samples
- ✅ Extract population genetics metrics
- ✅ Identify clinically relevant variants
- ✅ Perform quality control on variant datasets
- ✅ Handle complex inheritance patterns
- ✅ Process structural variants and copy number data
- ✅ Optimize performance for large VCF files
