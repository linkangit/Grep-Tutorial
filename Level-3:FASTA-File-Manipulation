# Level 3: FASTA File Manipulation

FASTA files are fundamental to bioinformatics. In this level, you'll master grep techniques specifically designed for analyzing and manipulating FASTA formatted sequences.

## Learning Objectives

By the end of this level, you will be able to:
- Extract sequences based on header patterns
- Count sequences and analyze sequence statistics
- Filter sequences by length and content
- Combine grep with other tools for advanced FASTA processing

## FASTA File Structure Refresher

```
>sequence_id description
ATCGATCGATCGATCG
GCTAGCTAGCTAGCTA
>another_sequence more_info
AAATTTCCCGGG
```

- Headers start with `>`
- Sequence data follows on subsequent lines
- Sequences may span multiple lines

## Core FASTA Manipulation Techniques

### 1. Basic Sequence Extraction

#### Extract All Headers
```bash
# Get all sequence identifiers
grep "^>" sequences.fasta

# Count total number of sequences
grep -c "^>" sequences.fasta

# Extract headers with line numbers
grep -n "^>" sequences.fasta
```

#### Extract Specific Sequences by Header Pattern
```bash
# Find sequences with "mitochondrial" in header
grep -i "mitochondrial" sequences.fasta

# Find sequences from specific organism
grep "Homo sapiens" sequences.fasta

# Extract sequences with gene symbols
grep -E "gene=[A-Z0-9]+" sequences.fasta
```

### 2. Advanced Header Analysis

#### Complex Header Parsing
```bash
# Find RefSeq sequences (NM_, NP_, etc.)
grep -E "^>[NX][MRP]_[0-9]+" sequences.fasta

# Extract Ensembl IDs
grep -E "^>ENS[A-Z]*[0-9]+" sequences.fasta

# Find sequences with version numbers
grep -E "^>.*\.[0-9]+" sequences.fasta

# Extract sequences from specific chromosomes in header
grep -E "chromosome [0-9XY]+" sequences.fasta
```

#### Extracting Header Components
```bash
# Get only the sequence IDs (first word after >)
grep "^>" sequences.fasta | grep -oE ">[A-Za-z0-9_]+"

# Extract gene names from headers
grep -oE "gene=[A-Za-z0-9_]+" sequences.fasta

# Find sequences with specific accession patterns
grep -E "^>gi\|[0-9]+\|" sequences.fasta
```

### 3. Sequence Content Analysis

#### Basic Sequence Pattern Matching
```bash
# Find sequences containing start codons
grep "ATG" sequences.fasta

# Find sequences with stop codons
grep -E "TAA|TAG|TGA" sequences.fasta

# Count sequences with poly-A tails (6 or more A's)
grep -c "A{6,}" sequences.fasta

# Find sequences with restriction enzyme sites
grep "GAATTC" sequences.fasta  # EcoRI
grep "AAGCTT" sequences.fasta  # HindIII
```

#### GC Content Patterns
```bash
# Find GC-rich regions (4 or more consecutive G or C)
grep -E "[GC]{4,}" sequences.fasta

# Find AT-rich regions
grep -E "[AT]{6,}" sequences.fasta

# Sequences with balanced nucleotide composition
grep -E "^[ATGC]*$" sequences.fasta
```

### 4. Sequence Length Analysis

#### Using Context Lines for Length Analysis
```bash
# Show sequence headers with their sequence lines
grep -A 1 "^>" sequences.fasta

# Find short sequences (assuming single-line sequences)
grep -A 1 "^>" sequences.fasta | grep -E "^[ATGC]{1,50}$"

# Find sequences longer than 100bp (single-line)
grep -A 1 "^>" sequences.fasta | grep -E "^[ATGC]{100,}$"
```

### 5. Multi-line Sequence Handling

FASTA sequences often span multiple lines. Here are techniques to handle them:

```bash
# Find headers of sequences containing specific patterns anywhere
grep -B 1 "GA
