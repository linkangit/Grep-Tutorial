# Level 4: FASTQ Quality Analysis

FASTQ files contain both sequence data and quality scores, making them essential for next-generation sequencing analysis. This level focuses on grep techniques for FASTQ quality control and analysis.

## Learning Objectives

By the end of this level, you will be able to:
- Navigate the 4-line FASTQ format structure
- Extract and analyze quality scores
- Filter reads based on quality criteria
- Detect and analyze adapter sequences
- Perform comprehensive FASTQ quality control

## FASTQ File Structure

Each FASTQ record consists of exactly 4 lines:

```
@read_id optional_description
ATCGATCGATCGATCGATCG
+optional_repeat_of_id
IIIIIIIIIIIIIIIIIIII
```

1. **Line 1**: Header starting with `@`
2. **Line 2**: DNA/RNA sequence
3. **Line 3**: Separator starting with `+` (may repeat read ID)
4. **Line 4**: Quality scores (same length as sequence)

## Quality Score Basics

- **Phred scores**: Q = -10 × log₁₀(P), where P is error probability
- **ASCII encoding**: Quality scores encoded as ASCII characters
- **Common ranges**: 
  - Phred+33 (Sanger): `!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`
  - Higher ASCII values = better quality

## Core FASTQ Analysis Techniques

### 1. Basic FASTQ Structure Analysis

#### Count Records and Validate Format
```bash
# Count total number of reads (divide by 4)
wc -l reads.fastq  # Then divide by 4

# Count reads using headers
grep -c "^@" reads.fastq

# Validate FASTQ format (check if every 4th line starts with @)
awk 'NR % 4 == 1 && !/^@/ {print "Invalid format at line " NR; exit 1}' reads.fastq
```

#### Extract FASTQ Components
```bash
# Extract all read IDs
grep "^@" reads.fastq

# Extract all sequences (every 2nd line starting from line 2)
awk 'NR % 4 == 2' reads.fastq

# Extract all quality lines (every 4th line starting from line 4)
awk 'NR % 4 == 0' reads.fastq

# Extract quality separators
grep "^+" reads.fastq
```

### 2. Read ID Analysis

#### Pattern Matching in Read IDs
```bash
# Find reads from specific sequencer/flowcell
grep "@HISEQ" reads.fastq
grep "@NOVASEQ" reads.fastq

# Extract reads with specific lane information
grep -E "@.*:1:" reads.fastq  # Lane 1

# Find paired-end read indicators
grep "/1$" reads.fastq  # Forward reads
grep "/2$" reads.fastq  # Reverse reads
grep "_1$" reads.fastq  # Alternative forward read format
```

#### Illumina Read ID Parsing
```bash
# Standard Illumina format: @instrument:run:flowcell:lane:tile:x:y read:filtered:control:index
# Extract specific components
grep -oE "@[^:]+:[^:]+:[^:]+:[^:]+:" reads.fastq  # First 4 fields

# Find reads from specific tiles
grep -E ":2101:" reads.fastq  # Tile 2101

# Extract index sequences
grep -oE ":[ATCGN]+$" reads.fastq
```

### 3. Quality Score Analysis

#### High-Quality Reads
```bash
# Find reads with high-quality scores (assuming Phred+33)
# ASCII 'I' = Q38, 'H' = Q37, etc.
grep -E "^[I-K]+$" reads.fastq  # Very high quality (Q38-42)

# Find reads with mostly high-quality scores
awk 'NR % 4 == 0 && /^[F-K]+$/' reads.fastq  # Q37-42

# Count high-quality reads
awk 'NR % 4 == 0 && /^[F-K]+$/ {count++} END {print count}' reads.fastq
```

#### Low-Quality Read Detection
```bash
# Find reads with low-quality scores
# ASCII '#' = Q2, '$' = Q3, etc.
grep -E "[!-/]" reads.fastq  # Quality scores Q0-14

# Find reads with many low-quality positions
awk 'NR % 4 == 0 && gsub(/[!-/]/, "") > 10' reads.fastq  # >10 low-quality positions

# Find reads ending with low quality (3' end degradation)
awk 'NR % 4 == 0 && /[!-/]{5,}$/' reads.fastq  # End with 5+ low-quality scores
```

#### Quality Distribution Analysis
```bash
# Find reads with specific quality patterns
awk 'NR % 4 == 0' reads.fastq | grep -E "^[A-K]{20,}[!-5]{10,}$"  # Good start, poor end

# Count reads by average quality (rough estimation)
awk 'NR % 4 == 0 && /^[I-K]+$/ {high++} /^[A-H]+$/ {medium++} /[!-@]/ {low++} END {print "High:" high " Medium:" medium " Low:" low}' reads.fastq
```

### 4. Sequence Content Analysis

#### GC Content and Composition
```bash
# Find GC-rich reads
awk 'NR % 4 == 2 && gsub(/[GC]/, "") / length > 0.6' reads.fastq

# Find AT-rich reads
awk 'NR % 4 == 2 && gsub(/[AT]/, "") / length > 0.7' reads.fastq

# Find reads with many N's (unknown nucleotides)
awk 'NR % 4 == 2 && gsub(/N/, "") > 5' reads.fastq
```

#### Adapter and Primer Detection
```bash
# Find reads containing common adapter sequences
grep "AGATCGGAAGAG" reads.fastq  # Illumina adapter
grep "CTGTCTCTTATA" reads.fastq  # Another common adapter

# Find reads with primer sequences
grep "GTTTCCCAGTCACGATA" reads.fastq  # Example primer

# Count adapter contamination
grep -c "AGATCGGAAGAG" reads.fastq
```

### 5. Advanced FASTQ Quality Control

#### Read Length Analysis
```bash
# Find reads of specific lengths (using sequence line)
awk 'NR % 4 == 2 && length == 150' reads.fastq  # Exactly 150 bp

# Find short reads (potential issues)
awk 'NR % 4 == 2 && length < 50' reads.fastq

# Find very long reads (potential concatenation issues)
awk 'NR % 4 == 2 && length > 300' reads.fastq
```

#### Complete Read Extraction with Context
```bash
# Extract complete high-quality reads (all 4 lines)
awk 'NR % 4 == 1 {header=$0} NR % 4 == 2 {seq=$0} NR % 4 == 3 {plus=$0} NR % 4 == 0 {if(/^[F-K]+$/) {print header; print seq; print plus; print}}' reads.fastq

# Extract reads with specific patterns and their quality
grep -A 3 "^@.*HiSeq" reads.fastq | grep -E "(^@|^[ATCGN]+$|^\+|^[!-~]+$)"
```

## Practical FASTQ Applications

### Example 1: Quality Control Pipeline
```bash
# Step 1: Count total reads
total_reads=$(grep -c "^@" reads.fastq)
echo "Total reads: $total_reads"

# Step 2: Count high-quality reads
high_qual=$(awk 'NR % 4 == 0 && !/[!-5]/ {count++} END {print count+0}' reads.fastq)
echo "High-quality reads: $high_qual"

# Step 3: Check for adapter contamination
adapter_count=$(grep -c "AGATCGGAAGAG" reads.fastq)
echo "Reads with adapters: $adapter_count"

# Step 4: Calculate quality percentage
echo "Quality percentage: $(echo "scale=2; $high_qual * 100 / $total_reads" | bc)%"
```

### Example 2: Read Filtering
```bash
# Create a high-quality subset
awk 'NR % 4 == 1 {header=$0} NR % 4 == 2 {seq=$0} NR % 4 == 3 {plus=$0} NR % 4 == 0 {qual=$0; if(qual !~ /[!-5]/ && seq !~ /N/) {print header; print seq; print plus; print qual}}' reads.fastq > high_quality_reads.fastq
```

### Example 3: Paired-End Analysis
```bash
# Check if reads are properly paired
forward_count=$(grep -c "/1$" reads_R1.fastq)
reverse_count=$(grep -c "/2$" reads_R2.fastq)
echo "Forward reads: $forward_count"
echo "Reverse reads: $reverse_count"

# Find unpaired reads
comm -23 <(grep "^@" reads_R1.fastq | sed 's|/1$||' | sort) <(grep "^@" reads_R2.fastq | sed 's|/2$||' | sort)
```

## Practice Exercises

### Exercise 1: Basic FASTQ Analysis
Using `sample_reads.fastq`:
1. Count the total number of reads
2. Extract all unique sequencer instruments used
3. Find the distribution of read lengths
4. Count reads with quality scores below Q20 (ASCII '5')

**Sample Solutions:**
```bash
# 1. Count reads
grep -c "^@" sample_reads.fastq

# 2. Extract instruments
grep "^@" sample_reads.fastq | cut -d: -f1 | sort -u

# 3. Read lengths (requires awk)
awk 'NR % 4 == 2 {print length}' sample_reads.fastq | sort -n | uniq -c

# 4. Low quality reads
awk 'NR % 4 == 0 && /[!-4]/ {count++} END {print count+0}' sample_reads.fastq
```

### Exercise 2: Quality Assessment
Using `sequencing_run.fastq`:
1. Find reads with average quality above Q30
2. Identify reads with adapter contamination
3. Count reads with more than 3 N's in the sequence
4. Extract reads from a specific lane (lane 2)

### Exercise 3: Advanced Filtering
Using `mixed_quality.fastq`:
1. Create a filtered file with only high-quality reads (no bases < Q25)
2. Find reads that start with high quality but degrade toward the 3' end
3. Identify potential PCR duplicates (identical sequences)
4. Extract reads with specific barcode patterns

## Real-World Quality Control Scenarios

### RNA-seq Quality Control
```bash
# Check for ribosomal RNA contamination
grep -i "ribosomal\|rrna\|18s\|28s" reads.fastq

# Find reads with polyA tails (mRNA enrichment check)
awk 'NR % 4 == 2 && /A{10,}$/' reads.fastq

# Check for strand-specific patterns
grep -E "^@.*strand" reads.fastq
```

### DNA-seq Quality Control
```bash
# Find reads with repetitive sequences
awk 'NR % 4 == 2 && /(A{10,}|T{10,}|G{10,}|C{10,})/' reads.fastq

# Check for mitochondrial DNA enrichment
grep -i "mitochondr\|chrM" reads.fastq

# Detect potential contamination
grep -E "(phix|phi.*x|vector)" reads.fastq -i
```

### Amplicon Sequencing
```bash
# Find reads with expected amplicon primers
grep "GTGCCAGCMGCCGCGGTAA" reads.fastq  # 16S V4 forward primer

# Check amplicon length distribution
awk 'NR % 4 == 2 && length >= 200 && length <= 300' reads.fastq | wc -l
```

## Quality Score Interpretation

| ASCII | Phred+33 | Error Rate | Quality |
|-------|----------|-----------|---------|
| ! | 0 | 100% | Very Poor |
| " | 1 | 79% | Poor |
| # | 2 | 63% | Poor |
| 5 | 20 | 1% | Fair |
| : | 25 | 0.3% | Good |
| ? | 30 | 0.1% | High |
| I | 40 | 0.01% | Very High |

## Common FASTQ Issues and Detection

| Issue | Detection Pattern | Example Command |
|-------|------------------|-----------------|
| Adapter contamination | Adapter sequences in reads | `grep "AGATCGGAAGAG" reads.fastq` |
| Quality degradation | Low scores at read ends | `awk 'NR%4==0 && /[!-/]{5,}$/'` |
| Short reads | Reads shorter than expected | `awk 'NR%4==2 && length<50'` |
| N content | Many unknown nucleotides | `awk 'NR%4==2 && gsub(/N/,"")>3'` |
| Duplicate reads | Identical sequence lines | `awk 'NR%4==2' \| sort \| uniq -d` |

## Tips for FASTQ Processing

1. **Memory considerations**: FASTQ files are large; use streaming when possible
2. **Quality encoding**: Verify Phred+33 vs Phred+64 encoding
3. **Paired-end coordination**: Keep forward and reverse reads synchronized
4. **Preprocessing**: Consider quality trimming before analysis
5. **Tool integration**: Combine with FastQC, Trimmomatic, etc.

## Next Steps

You're ready for Level 5, where we'll explore advanced regular expressions including grouping, backreferences, and complex pattern combinations that will take your grep skills to the next level!

## Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `grep -c "^@" file.fastq` | Count reads | Total read count |
| `awk 'NR % 4 == 2'` | Extract sequences | Get all sequence lines |
| `awk 'NR % 4 == 0'` | Extract quality | Get all quality lines |
| `grep -E ":[ATCGN]+$"` | Extract barcodes | Find index sequences |
| `awk 'NR%4==0 && !/[!-5]/'` | High quality reads | Find good quality |
| `grep "AGATCGGAAGAG"` | Find adapters | Detect contamination |
