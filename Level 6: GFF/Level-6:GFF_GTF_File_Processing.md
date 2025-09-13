# Level 6: GFF/GTF File Processing

GFF (General Feature Format) and GTF (Gene Transfer Format) files are essential for genomic annotation. This level focuses on parsing these complex tab-delimited files using advanced grep techniques.

## Learning Objectives

By the end of this level, you will be able to:
- Parse GFF/GTF file structures and extract specific fields
- Filter features by genomic coordinates and attributes
- Extract complex attribute information using regex
- Perform coordinate-based queries and overlaps
- Analyze gene structures and alternative splicing

## GFF/GTF File Structure

### GFF3 Format (9 tab-separated columns):
```
chr1    RefSeq    gene      1000    9000    .    +    .    ID=gene1;Name=BRCA1;biotype=protein_coding
chr1    RefSeq    mRNA      1000    9000    .    +    .    ID=transcript1;Parent=gene1;Name=BRCA1-001
chr1    RefSeq    exon      1000    1200    .    +    .    ID=exon1;Parent=transcript1
chr1    RefSeq    CDS       1050    1200    .    +    0    ID=cds1;Parent=transcript1
```

### GTF Format (similar structure, different attributes):
```
chr1    RefSeq    gene      1000    9000    .    +    .    gene_id "ENSG001"; gene_name "BRCA1"; gene_biotype "protein_coding";
chr1    RefSeq    transcript 1000   9000    .    +    .    gene_id "ENSG001"; transcript_id "ENST001"; gene_name "BRCA1";
```

**Columns:**
1. **seqname** (chromosome)
2. **source** (annotation source)
3. **feature** (gene, exon, CDS, etc.)
4. **start** (1-based start position)
5. **end** (end position)
6. **score** (quality score or .)
7. **strand** (+ or -)
8. **frame** (0, 1, 2, or .)
9. **attribute** (key-value pairs)

## Basic GFF/GTF Parsing

### 1. Feature Type Filtering

#### Basic Feature Extraction
```bash
# Extract all genes
grep -E "^[^#].*\tgene\t" annotations.gff

# Get all exons
grep -E "\texon\t" annotations.gff

# Find CDS features
grep -E "\tCDS\t" annotations.gtf

# Extract mRNA/transcript features
grep -E "\t(mRNA|transcript)\t" annotations.gff
```

#### Multiple Feature Types
```bash
# Get genes and transcripts
grep -E "\t(gene|transcript|mRNA)\t" annotations.gff

# Extract all coding features
grep -E "\t(gene|mRNA|exon|CDS)\t" annotations.gff

# Find regulatory features
grep -E "\t(promoter|enhancer|silencer)\t" regulatory.gff
```

### 2. Chromosome and Coordinate Filtering

#### Chromosome-Specific Queries
```bash
# Single chromosome
grep -E "^chr1\t" annotations.gff

# Multiple specific chromosomes
grep -E "^(chr1|chr2|chr3)\t" annotations.gff

# Autosomes only (chr1-22)
grep -E "^chr([1-9]|1[0-9]|2[0-2])\t" annotations.gff

# Sex chromosomes
grep -E "^(chrX|chrY)\t" annotations.gff

# All standard chromosomes (excluding patches, random)
grep -E "^chr([1-9]|1[0-9]|2[0-2]|X|Y)\t" annotations.gff
```

#### Coordinate Range Filtering
```bash
# Features starting after position 1000000
awk '$4 > 1000000' annotations.gff

# Features ending before position 2000000  
awk '$5 < 2000000' annotations.gff

# Features within a specific range (1MB-2MB on chr1)
grep "^chr1" annotations.gff | awk '$4 >= 1000000 && $5 <= 2000000'

# Large features (>100kb)
awk '$5 - $4 > 100000' annotations.gff

# Small features (<1kb)
awk '$5 - $4 < 1000' annotations.gff
```

### 3. Strand-Specific Analysis

```bash
# Forward strand features
grep -E "\t\+\t" annotations.gff

# Reverse strand features  
grep -E "\t-\t" annotations.gff

# Strand-specific genes on chromosome 1
grep "^chr1" annotations.gff | grep -E "\tgene\t" | grep "\+"
```

## Advanced Attribute Parsing

### 4. GFF3 Attribute Extraction

#### Key-Value Pair Parsing
```bash
# Extract gene IDs
grep -oE "ID=[^;]+" annotations.gff

# Get gene names
grep -oE "Name=[^;]+" annotations.gff

# Extract biotype information
grep -oE "biotype=[^;]+" annotations.gff

# Find parent relationships
grep -oE "Parent=[^;]+" annotations.gff
```

#### Complex Attribute Queries
```bash
# Genes with specific biotype
grep "biotype=protein_coding" annotations.gff

# Find pseudogenes
grep "biotype=.*pseudo" annotations.gff

# Extract genes with specific gene names
grep -E "Name=(BRCA1|BRCA2|TP53)" annotations.gff

# Find features with multiple parents
grep -E "Parent=[^;]*,[^;]*" annotations.gff
```

### 5. GTF Attribute Parsing

#### GTF-Specific Patterns
```bash
# Extract gene IDs from GTF
grep -oE 'gene_id "[^"]+"' annotations.gtf

# Get transcript IDs
grep -oE 'transcript_id "[^"]+"' annotations.gtf

# Extract gene names from GTF
grep -oE 'gene_name "[^"]+"' annotations.gtf

# Find specific gene biotypes
grep 'gene_biotype "protein_coding"' annotations.gtf
```

#### Complex GTF Queries
```bash
# Find genes with specific attributes combination
grep 'gene_biotype "protein_coding"' annotations.gtf | grep 'gene_name "BRCA1"'

# Extract transcript support levels
grep -oE 'transcript_support_level "[^"]+"' annotations.gtf

# Find manually annotated transcripts
grep 'tag "basic"' annotations.gtf
```

## Genomic Analysis Applications

### Example 1: Gene Structure Analysis

#### Analyze Gene Components
```bash
# Count different feature types
grep -E "^[^#]" annotations.gff | cut -f3 | sort | uniq -c

# Find genes with multiple transcripts
grep -E "\tgene\t" annotations.gff | \
grep -oE "ID=[^;]+" | \
while read gene; do
    count=$(grep "Parent=${gene#ID=}" annotations.gff | wc -l)
    if [ $count -gt 1 ]; then echo "$gene has $count transcripts"; fi
done
```

#### Exon Analysis
```bash
# Count exons per transcript
grep -E "\texon\t" annotations.gff | grep -oE "Parent=[^;]+" | sort | uniq -c

# Find single-exon genes
grep -E "\texon\t" annotations.gff | grep -oE "Parent=[^;]+" | sort | uniq -c | awk '$1 == 1'

# Large exons (>10kb)
grep -E "\texon\t" annotations.gff | awk '$5 - $4 > 10000'
```

### Example 2: Alternative Splicing Analysis

```bash
# Find genes with multiple transcript variants
grep -E "\ttranscript\t" annotations.gff | grep -oE "Parent=[^;]+" | sort | uniq -c | awk '$1 > 1'

# Identify alternative first exons
grep -E "\texon\t.*exon_number=1" annotations.gtf | grep -oE 'gene_id "[^"]+"' | sort | uniq -c | awk '$1 > 1'

# Find retained introns (exons overlapping intron regions)
# This requires more complex analysis combining multiple features
```

### Example 3: Regulatory Element Analysis

```bash
# Extract promoter regions (typically upstream of TSS)
grep -E "\tpromoter\t" regulatory.gff

# Find enhancers on specific chromosomes
grep -E "^chr1\t.*enhancer" regulatory.gff

# CpG islands analysis
grep -E "CpG.*island" annotations.gff

# TFBS (Transcription Factor Binding Sites)
grep -E "TFBS|binding_site" regulatory.gff
```

### Example 4: Coordinate Overlap Analysis

```bash
# Find features overlapping a specific region (chr1:1000000-2000000)
grep "^chr1" annotations.gff | awk '($4 <= 2000000 && $5 >= 1000000)'

# Features entirely within a region
grep "^chr1" annotations.gff | awk '($4 >= 1000000 && $5 <= 2000000)'

# Features spanning a specific position
grep "^chr1" annotations.gff | awk '($4 <= 1500000 && $5 >= 1500000)'
```

## Advanced GFF/GTF Processing

### 6. Hierarchical Relationship Analysis

#### Parent-Child Relationships
```bash
# Find all children of a specific gene
gene_id="gene001"
grep "Parent=${gene_id}" annotations.gff

# Build gene structure hierarchy
grep -E "ID=gene001" annotations.gff  # Gene
grep -E "Parent=gene001" annotations.gff  # Transcripts
grep -E "Parent=transcript001" annotations.gff  # Exons/CDS
```

#### Complex Hierarchical Queries
```bash
# Find genes with CDS but no UTR annotations
gene_with_cds=$(grep -E "\tCDS\t" annotations.gff | grep -oE "Parent=[^;]+" | sort -u)
gene_with_utr=$(grep -E "\tUTR\t" annotations.gff | grep -oE "Parent=[^;]+" | sort -u)
comm -23 <(echo "$gene_with_cds") <(echo "$gene_with_utr")
```

### 7. Cross-Reference Analysis

```bash
# Find features with external database references
grep -E "(Dbxref|dbxref)=" annotations.gff

# Extract specific database cross-references
grep -oE "Dbxref=[^;]*HGNC:[^;,]*" annotations.gff

# Find Ensembl cross-references
grep -E "Dbxref=[^;]*ENS[GT][0-9]+" annotations.gff
```

### 8. Quality and Completeness Analysis

```bash
# Find features missing essential attributes
grep -E "\tgene\t" annotations.gff | grep -v "Name="  # Genes without names

# Identify truncated features (suspicious coordinates)
awk '$4 == $5' annotations.gff  # Start equals end

# Find features with unusual coordinates
awk '$4 > $5 {print "Invalid coordinates: " $0}' annotations.gff

# Check for strand consistency in gene families
grep -E "ID=gene.*" annotations.gff | grep -oE "(ID=[^;]+|[+-])" | paste - -
```

## Practice Exercises

### Exercise 1: Basic Feature Analysis
Using `genome_annotations.gff`:
1. Count the total number of genes, transcripts, and exons
2. Find all protein-coding genes on chromosome 1
3. Extract genes on the negative strand
4. List all different feature types in the file

**Sample Solutions:**
```bash
# 1. Count features
grep -c -E "\tgene\t" genome_annotations.gff
grep -c -E "\ttranscript\t" genome_annotations.gff  
grep -c -E "\texon\t" genome_annotations.gff

# 2. Protein-coding genes on chr1
grep "^chr1" genome_annotations.gff | grep -E "\tgene\t" | grep "protein_coding"

# 3. Negative strand genes
grep -E "\tgene\t.*\t-\t" genome_annotations.gff

# 4. Feature types
grep -E "^[^#]" genome_annotations.gff | cut -f3 | sort -u
```

### Exercise 2: Attribute Parsing
Using `detailed_annotations.gtf`:
1. Extract all unique gene names
2. Find genes with transcript support level 1
3. Count transcripts per gene
4. Identify genes with alternative splicing (multiple transcripts)

### Exercise 3: Coordinate Analysis
Using `chromosome_features.gff`:
1. Find the largest gene on each chromosome
2. Identify overlapping genes on opposite strands
3. Extract features within the first 10Mb of each chromosome
4. Find genes that span more than 1Mb

## Real-World Applications

### Comparative Genomics
```bash
# Compare gene density across chromosomes
for chr in $(seq 1 22) X Y; do
    echo -n "chr$chr: "
    grep "^chr$chr" annotations.gff | grep -c -E "\tgene\t"
done

# Find orthologous gene pairs
grep -E "ortholog=" annotations.gff | grep -oE "(ID=[^;]+|ortholog=[^;]+)"
```

### Functional Analysis
```bash
# Extract genes by functional category
grep "GO:0003677" annotations.gff  # DNA binding
grep "GO:0016740" annotations.gff  # Transferase activity

# Find disease-associated genes
grep -iE "(disease|pathogen|disorder)" annotations.gff

# Identify druggable targets
grep -iE "(drug|target|therapeutic)" annotations.gff
```

### Structural Variation Analysis
```bash
# Find segmental duplications
grep -E "segmental.*dup" annotations.gff

# Identify repeat regions
grep -E "(repeat|transpos)" annotations.gff

# Find structural variants
grep -E "(inversion|deletion|insertion)" variants.gff
```

## File Format Conversion and Integration

```bash
# Convert GFF3 to simple BED format
grep -E "\tgene\t" annotations.gff | awk -F'\t' '{print $1"\t"($4-1)"\t"$5"\t"$9"\t0\t"$7}'

# Extract GTF gene information for other tools
grep -E "\tgene\t" annotations.gtf | grep -oE '(gene_id "[^"]+"|gene_name "[^"]+")'

# Create gene coordinate list
grep -E "\tgene\t" annotations.gff | awk -F'\t' '{print $1":"$4"-"$5}'
```

## Performance Optimization for Large Files

```bash
# Use field-specific patterns to avoid full regex when possible
grep -E "^chr1\t.*\tgene\t" annotations.gff  # More efficient than multiple greps

# Index-based processing for repeated queries
sort -k1,1 -k4,4n annotations.gff > sorted_annotations.gff

# Memory-efficient streaming for large files
grep -E "\tgene\t" huge_annotations.gff | head -1000  # Process subset first
```

## Common GFF/GTF Patterns

| Pattern | Description | Example Usage |
|---------|-------------|---------------|
| `^chr[0-9XY]+\t` | Chromosome filter | Extract autosomal features |
| `\t(gene\|transcript)\t` | Feature types | Get gene structure |
| `ID=[^;]+` | Extract GFF3 IDs | Parse identifiers |
| `gene_id "[^"]+""` | Extract GTF gene IDs | Parse GTF attributes |
| `\t[+-]\t` | Strand information | Strand-specific analysis |
| `$4 >= [0-9]+ && $5 <= [0-9]+` | Coordinate ranges | Region-based queries |

## Tips for GFF/GTF Processing

1. **Understand format differences**: GFF3 vs GTF attribute syntax
2. **Handle header lines**: Use `grep -v "^#"` to skip comments
3. **Field consistency**: Always validate tab-delimited structure
4. **Coordinate systems**: GFF/GTF uses 1-based coordinates
5. **Attribute parsing**: Different escaping rules for GFF3 vs GTF
6. **Performance**: Use awk for numeric comparisons, grep for pattern matching

## Troubleshooting Common Issues

### Format Validation
```bash
# Check for proper tab separation (should be 8 tabs per line)
awk -F'\t' 'NF != 9 {print "Line " NR " has " NF " fields: " $0}' annotations.gff

# Validate coordinate consistency
awk '$4 > $5 {print "Invalid coordinates at line " NR ": " $0}' annotations.gff

# Check strand field validity
grep -E "[^+-.]" annotations.gff | cut -f7 | sort -u  # Should be empty

# Find malformed attribute fields
grep -E "\t[^;]*=[^;]*[^;]$" annotations.gff | head -5  # Check attribute syntax
```

### Data Quality Checks
```bash
# Find duplicate feature IDs
grep -oE "ID=[^;]+" annotations.gff | sort | uniq -d

# Check for missing parent references
orphans=$(grep -oE "Parent=[^;]+" annotations.gff | sed 's/Parent=//' | sort -u)
parents=$(grep -oE "ID=[^;]+" annotations.gff | sed 's/ID=//' | sort -u)
comm -23 <(echo "$orphans") <(echo "$parents")

# Identify unusual feature types
cut -f3 annotations.gff | sort | uniq -c | sort -rn
```

## Advanced Integration Examples

### Combining Multiple Annotation Sources
```bash
# Merge RefSeq and Ensembl annotations
grep "RefSeq" annotations.gff > refseq_features.gff
grep "Ensembl" annotations.gff > ensembl_features.gff

# Find features present in both sources
grep -E "gene_name=[^;]+" refseq_features.gff | grep -oE "gene_name=[^;]+" | sort > refseq_genes.txt
grep -E "gene_name=[^;]+" ensembl_features.gff | grep -oE "gene_name=[^;]+" | sort > ensembl_genes.txt
comm -12 refseq_genes.txt ensembl_genes.txt  # Common genes
```

### Creating Custom Analysis Pipelines
```bash
# Multi-step gene analysis pipeline
analyze_gene() {
    local gene_name=$1
    echo "Analyzing gene: $gene_name"
    
    # Extract gene coordinates
    gene_coords=$(grep -E "\tgene\t.*Name=${gene_name};" annotations.gff | cut -f1,4,5)
    echo "Coordinates: $gene_coords"
    
    # Count transcripts
    transcript_count=$(grep -E "\ttranscript\t.*Parent=.*${gene_name}" annotations.gff | wc -l)
    echo "Transcripts: $transcript_count"
    
    # Count exons
    exon_count=$(grep -E "\texon\t" annotations.gff | grep -E "Parent=.*${gene_name}" | wc -l)
    echo "Total exons: $exon_count"
}

# Usage: analyze_gene "BRCA1"
```

### Statistical Analysis
```bash
# Gene length distribution
grep -E "\tgene\t" annotations.gff | awk '{print $5-$4+1}' | sort -n > gene_lengths.txt

# Exon count per gene statistics
gene_exon_counts() {
    grep -E "\tgene\t" annotations.gff | while read line; do
        gene_id=$(echo "$line" | grep -oE "ID=[^;]+" | sed 's/ID=//')
        exon_count=$(grep -E "\texon\t.*Parent=.*${gene_id}" annotations.gff | wc -l)
        echo "$gene_id $exon_count"
    done
}

# Strand distribution analysis
echo "Forward strand genes: $(grep -E "\tgene\t.*\t\+\t" annotations.gff | wc -l)"
echo "Reverse strand genes: $(grep -E "\tgene\t.*\t-\t" annotations.gff | wc -l)"
```

## Real-World Case Studies

### Case Study 1: Cancer Gene Analysis
```bash
# Extract known cancer genes
cancer_genes="TP53|BRCA1|BRCA2|MYC|RAS|APC|RB1|PTEN"
grep -iE "Name=(${cancer_genes});" annotations.gff > cancer_gene_annotations.gff

# Analyze their chromosomal distribution
cut -f1 cancer_gene_annotations.gff | sort | uniq -c

# Check for alternative splicing in cancer genes
cancer_transcripts=$(grep -E "\ttranscript\t" cancer_gene_annotations.gff | wc -l)
cancer_genes_count=$(grep -E "\tgene\t" cancer_gene_annotations.gff | wc -l)
echo "Average transcripts per cancer gene: $(echo "$cancer_transcripts / $cancer_genes_count" | bc -l)"
```

### Case Study 2: Housekeeping Gene Identification
```bash
# Find widely expressed genes (present in multiple tissues)
housekeeping_patterns="ribosom|actin|tubulin|histone|ubiquit"
grep -iE "${housekeeping_patterns}" annotations.gff | grep -E "\tgene\t"

# Analyze their genomic features
grep -iE "${housekeeping_patterns}" annotations.gff | grep -E "\tgene\t" | \
awk '{print $1 "\t" $5-$4+1 "\t" $7}' | \
sort -k2,2n  # Sort by gene length
```

### Case Study 3: Pseudogene Analysis
```bash
# Extract pseudogenes
grep "biotype=.*pseudo" annotations.gff > pseudogenes.gff

# Compare with functional genes
functional_genes=$(grep "biotype=protein_coding" annotations.gff | wc -l)
pseudogene_count=$(wc -l < pseudogenes.gff)
echo "Pseudogene ratio: $(echo "scale=3; $pseudogene_count / $functional_genes" | bc)"

# Find processed vs unprocessed pseudogenes
grep "processed_pseudogene" pseudogenes.gff | wc -l
grep "unprocessed_pseudogene" pseudogenes.gff | wc -l
```

## Quick Reference - GFF/GTF Patterns

| Task | Pattern | Example |
|------|---------|---------|
| Extract genes | `\tgene\t` | `grep -E "\tgene\t" file.gff` |
| Chromosome filter | `^chr[0-9XY]+\t` | `grep -E "^chr1\t" file.gff` |
| Feature coordinates | `$4` and `$5` | `awk '$4 > 1000000' file.gff` |
| Strand filtering | `\t[+-]\t` | `grep -E "\t\+\t" file.gff` |
| Attribute extraction | `ID=[^;]+` | `grep -oE "ID=[^;]+" file.gff` |
| GTF gene ID | `gene_id "[^"]+"` | `grep -oE 'gene_id "[^"]+"' file.gtf` |
| Parent relationships | `Parent=[^;]+` | `grep -oE "Parent=[^;]+" file.gff` |
| Biotype filtering | `biotype=[^;]+` | `grep "biotype=protein_coding" file.gff` |

## Command Combinations for Complex Queries

```bash
# Multi-condition filtering
grep "^chr1" annotations.gff | \
grep -E "\tgene\t" | \
grep "biotype=protein_coding" | \
awk '$5 - $4 > 50000'  # Large protein-coding genes on chr1

# Hierarchical feature extraction
gene_id="ENSG00000139618"  # BRCA2
grep "ID=${gene_id}" annotations.gff  # Gene
grep "Parent=${gene_id}" annotations.gff  # Transcripts
grep "Parent=" annotations.gff | grep -E "(${gene_id}|transcript.*${gene_id})"  # All children

# Cross-format compatibility
# Convert GTF to GFF3-like format for consistent processing
sed 's/gene_id "\([^"]*\)"/ID=\1/g; s/transcript_id "\([^"]*\)"/Parent=\1/g' annotations.gtf
```

## Next Steps

Congratulations! You've now mastered advanced GFF/GTF processing with grep. You're ready for Level 7, where we'll tackle VCF (Variant Call Format) files and learn how to analyze genetic variants, genotypes, and population genomics data using sophisticated grep techniques!

## Summary of Level 6 Skills

- ✅ Parse complex GFF/GTF file structures
- ✅ Extract and filter genomic features by multiple criteria
- ✅ Handle hierarchical gene-transcript-exon relationships
- ✅ Perform coordinate-based genomic queries
- ✅ Analyze alternative splicing and gene structure
- ✅ Integrate multiple annotation sources
- ✅ Validate and troubleshoot annotation files
- ✅ Build complex genomic analysis pipelines
