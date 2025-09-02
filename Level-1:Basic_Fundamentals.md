# Level 1: Basic Grep Fundamentals

Welcome to your first grep tutorial! In this level, you'll learn the essential grep commands that form the foundation of text searching in bioinformatics.

## Learning Objectives

By the end of this level, you will be able to:
- Perform basic text searches in biological files
- Use case-insensitive searches
- Count matches and display line numbers
- Search for whole words only

## What is Grep?

`grep` (Global Regular Expression Print) is a command-line tool for searching text patterns in files. It's essential for bioinformaticians working with large datasets where manual searching is impractical.

## Basic Syntax

```bash
grep [options] "pattern" filename
```

## Core Commands

### 1. Simple Text Search

Search for a specific string in a file:

```bash
grep "ATCG" sequences.fasta
```

**Example Output:**
```
ATCGATCGATCGATCG
GATCGATCGATCGATCG
```

### 2. Case-Insensitive Search (`-i`)

Ignore case when searching:

```bash
grep -i "atcg" sequences.fasta
```

This will match "ATCG", "atcg", "AtCg", etc.

### 3. Count Matches (`-c`)

Count the number of lines containing the pattern:

```bash
grep -c ">" sequences.fasta
```

This counts FASTA headers (useful for counting sequences).

### 4. Display Line Numbers (`-n`)

Show line numbers where matches occur:

```bash
grep -n "mitochondrial" genes.gff
```

**Example Output:**
```
45: chr1    RefSeq  gene    1000    2000    .   +   .   ID=gene1;Name=mitochondrial_gene1
78: chr2    RefSeq  gene    3000    4000    .   -   .   ID=gene2;Name=mitochondrial_gene2
```

### 5. Whole Word Search (`-w`)

Match complete words only:

```bash
grep -w "ATP" proteins.txt
```

This matches "ATP" but not "ATPase" or "mATP".

### 6. Show Only Matching Text (`-o`)

Display only the matched portion:

```bash
grep -o "chr[0-9]*" coordinates.bed
```

**Example Output:**
```
chr1
chr2
chr10
chr22
```

## Bioinformatics Examples

### Example 1: Finding Specific Genes

Search for insulin genes in a GFF file:

```bash
grep -i "insulin" human_genes.gff
```

### Example 2: Counting Sequences in FASTA

Count the number of sequences in a FASTA file:

```bash
grep -c "^>" sequences.fasta
```

The `^` symbol means "start of line" (we'll cover this more in Level 2).

### Example 3: Finding Quality Scores in FASTQ

Search for high-quality reads (quality line starting with high scores):

```bash
grep -n "^+$" reads.fastq
```

### Example 4: Extracting Chromosome Information

Find all entries for chromosome 21:

```bash
grep -w "chr21" variants.vcf
```

## Practice Exercises

### Exercise 1: Basic Search
Using the provided `sample_genes.fasta` file:
1. Search for the sequence "GAATTC" (EcoRI restriction site)
2. Count how many sequences contain this pattern
3. Find the line numbers where this pattern occurs

### Exercise 2: Case-Insensitive Gene Search
Using `gene_annotations.gff`:
1. Search for "insulin" genes (case-insensitive)
2. Search for "ATP" related genes
3. Count how many mitochondrial genes are present

### Exercise 3: Chromosome Analysis
Using `genome_coordinates.bed`:
1. Find all entries for chromosome X
2. Count entries for each chromosome (1-22, X, Y)
3. Extract only the chromosome names from the file

## Sample Commands for Practice

```bash
# Search for restriction enzyme sites
grep "GAATTC" sequences.fasta

# Count sequences in FASTA file
grep -c "^>" sequences.fasta

# Find genes on specific chromosome
grep -w "chr7" annotations.gff

# Case-insensitive search for gene names
grep -i "insulin" gene_list.txt

# Show line numbers for matches
grep -n "mitochondrial" features.gff
```

## Common Beginner Mistakes

1. **Forgetting quotes**: Always quote your search pattern
   - ❌ `grep ATCG file.txt`
   - ✅ `grep "ATCG" file.txt`

2. **Case sensitivity**: Remember that grep is case-sensitive by default
   - Use `-i` for case-insensitive searches

3. **Partial matches**: grep finds partial matches by default
   - Use `-w` for whole word matches when needed

## Tips for Success

- Start with simple patterns before moving to complex ones
- Use `-n` to see where matches occur in your files
- Combine options: `grep -in "pattern" file` (case-insensitive with line numbers)
- Practice with small files first to understand the output

## Next Steps

Once you're comfortable with these basic commands, you're ready for Level 2, where we'll explore regular expressions and pattern matching that will dramatically expand your search capabilities!

## Quick Reference

| Option | Description | Example |
|--------|-------------|---------|
| `-i` | Case-insensitive | `grep -i "atcg" file` |
| `-c` | Count matches | `grep -c ">" file.fasta` |
| `-n` | Show line numbers | `grep -n "gene" file.gff` |
| `-w` | Whole words only | `grep -w "ATP" file` |
| `-o` | Show only matches | `grep -o "chr[0-9]*" file` |
