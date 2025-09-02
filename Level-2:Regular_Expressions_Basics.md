# Level 2: Regular Expressions Basics

Now that you've mastered basic grep, it's time to unlock its true power with regular expressions (regex). Regular expressions allow you to search for patterns rather than just fixed text.

## Learning Objectives

By the end of this level, you will be able to:
- Use character classes and ranges for flexible matching
- Apply anchors to match patterns at specific positions
- Utilize quantifiers to match repeated patterns
- Combine multiple regex elements for complex searches

## What are Regular Expressions?

Regular expressions are patterns that describe sets of strings. Instead of searching for exact text, you can search for patterns like "any DNA sequence starting with ATG" or "any gene ID that ends with numbers."

## Essential Regex Components

### 1. Character Classes

#### Basic Character Classes
- `.` - Matches any single character
- `[ABC]` - Matches any one of A, B, or C
- `[^ABC]` - Matches any character EXCEPT A, B, or C

```bash
# Find any nucleotide followed by TCG
grep ".TCG" sequences.fasta

# Match A, T, G, or C (any nucleotide)
grep "[ATGC]" sequences.fasta

# Match anything except N (unknown nucleotide)
grep "[^N]" sequences.fasta
```

#### Predefined Character Classes
- `[0-9]` - Any digit
- `[A-Z]` - Any uppercase letter
- `[a-z]` - Any lowercase letter
- `[A-Za-z]` - Any letter
- `[A-Za-z0-9]` - Any letter or digit

```bash
# Find gene IDs with numbers
grep "gene[0-9]" annotations.gff

# Match chromosome names (chr followed by numbers)
grep "chr[0-9]" coordinates.bed

# Find protein sequences with any amino acid
grep "[A-Z]" proteins.fasta
```

### 2. Anchors

Anchors specify where in the line the pattern should match:

- `^` - Beginning of line
- `$` - End of line

```bash
# Find FASTA headers (lines starting with >)
grep "^>" sequences.fasta

# Find sequences ending with stop codon
grep "TAG$" sequences.fasta

# Find empty lines
grep "^$" file.txt

# Find lines that contain only ATGC
grep "^[ATGC]*$" sequences.fasta
```

### 3. Quantifiers

Quantifiers specify how many times a pattern should repeat:

- `*` - Zero or more times
- `+` - One or more times (requires `-E` flag)
- `?` - Zero or one time (requires `-E` flag)
- `{n}` - Exactly n times (requires `-E` flag)
- `{n,m}` - Between n and m times (requires `-E` flag)

```bash
# Find sequences with multiple A's
grep "A*" sequences.fasta

# Find gene IDs with one or more digits (extended regex)
grep -E "gene[0-9]+" annotations.gff

# Find optional chromosome prefix
grep -E "chr[0-9]?" coordinates.bed

# Find exactly 3 nucleotides
grep -E "[ATGC]{3}" sequences.fasta

# Find sequences 20-30 nucleotides long
grep -E "^[ATGC]{20,30}$" sequences.fasta
```

## Extended Regular Expressions (`-E` flag)

The `-E` flag enables extended regular expressions, giving you access to more powerful features:

```bash
# Use + quantifier (one or more)
grep -E "A+" sequences.fasta

# Use ? quantifier (zero or one)
grep -E "ATG?" sequences.fasta

# Use {} for specific counts
grep -E "[ATGC]{50}" sequences.fasta
```

## Bioinformatics Applications

### Example 1: Finding Start Codons

Find sequences that start with a start codon (ATG):

```bash
grep "^ATG" cds_sequences.fasta
```

### Example 2: Identifying Restriction Sites

Find EcoRI sites (GAATTC) with possible variations:

```bash
# Exact match
grep "GAATTC" sequences.fasta

# With possible single nucleotide variations
grep -E "G[ATGC]ATTC" sequences.fasta
```

### Example 3: Extracting Gene IDs

Find gene IDs that follow a specific pattern:

```bash
# Gene IDs like GENE001, GENE002, etc.
grep -E "GENE[0-9]{3}" gene_list.txt

# More flexible: GENE followed by any number of digits
grep -E "GENE[0-9]+" gene_list.txt
```

### Example 4: Quality Control in FASTQ

Find FASTQ quality lines (start with + and may contain sequence ID):

```bash
# Simple quality line indicator
grep "^+" reads.fastq

# Quality line with optional sequence ID
grep -E "^\+.*" reads.fastq
```

### Example 5: Chromosome Validation

Find properly formatted chromosome entries:

```bash
# Standard chromosomes (chr1-22, chrX, chrY)
grep -E "^chr([1-9]|1[0-9]|2[0-2]|X|Y)" genome.bed

# Any chromosome followed by coordinates
grep -E "chr[0-9XY]+\s+[0-9]+" coordinates.bed
```

## Practice Exercises

### Exercise 1: DNA Pattern Recognition

Using `dna_sequences.fasta`:
1. Find all sequences starting with ATG (start codon)
2. Find sequences ending with any stop codon (TAA, TAG, TGA)
3. Count sequences that contain exactly 5 consecutive A's

**Solution hints:**
```bash
# 1. Start codon
grep "^ATG" dna_sequences.fasta

# 2. Stop codons (you'll need alternation - we'll cover this more later)
grep -E "(TAA|TAG|TGA)$" dna_sequences.fasta

# 3. Exactly 5 A's
grep -E "A{5}" dna_sequences.fasta
```

### Exercise 2: Gene Annotation Analysis

Using `gene_features.gff`:
1. Find all entries for chromosomes 1-9 (single digit)
2. Extract gene features that have exactly 4-digit IDs
3. Find features that start at position 1000 or higher

### Exercise 3: Quality Score Patterns

Using `quality_data.fastq`:
1. Find quality lines that contain only high-quality scores (assume scores I and above)
2. Count sequences with IDs containing exactly 8 characters after "@"
3. Find sequences where the quality line matches the sequence ID pattern

## Advanced Examples

### Combining Multiple Patterns

```bash
# Find sequences that start with ATG and end with stop codon
grep -E "^ATG.*TA[AG]$" sequences.fasta

# Find gene IDs with specific chromosome pattern
grep -E "chr[1-9]_gene[0-9]{4}" annotations.gff

# Validate FASTA header format
grep -E "^>.*\|.*\|" sequences.fasta
```

### Real-world Bioinformatics Use Cases

```bash
# Find splice sites (GT...AG pattern in introns)
grep -E "GT[ATGC]*AG" intron_sequences.fasta

# Identify potential ORFs (ATG to stop codon)
grep -E "ATG[ATGC]*(TAA|TAG|TGA)" sequences.fasta

# Extract specific chromosome regions
grep -E "chr[0-9]+:[0-9]+-[0-9]+" region_list.txt

# Find conserved domains with pattern
grep -E "[RK].[ST]" protein_sequences.fasta
```

## Common Regex Patterns for Bioinformatics

| Pattern | Description | Example |
|---------|-------------|---------|
| `^ATG` | Start codon at beginning | Start of coding sequence |
| `TA[AG]$` | Stop codons at end | End of coding sequence |
| `[ATGC]{20,}` | At least 20 nucleotides | Minimum sequence length |
| `^>.*` | FASTA headers | Sequence identifiers |
| `chr[0-9XY]+` | Chromosome identifiers | Genomic coordinates |
| `[0-9]+\.[0-9]+` | Version numbers | Gene/transcript versions |

## Tips and Best Practices

1. **Start simple**: Build complex patterns step by step
2. **Test incrementally**: Test each component before combining
3. **Use quotes**: Always quote your regex patterns
4. **Escape special characters**: Use `\` to treat special characters literally
5. **Use `-E` for extended features**: Enable advanced quantifiers and grouping

## Common Mistakes to Avoid

1. **Forgetting anchors**: `grep "ATG"` vs `grep "^ATG"`
2. **Missing `-E` flag**: Required for +, ?, {}, and other extended features
3. **Unescaped special characters**: `.` matches any character, not literal dot
4. **Greedy matching**: `.*` can match more than expected

## Quick Reference

| Symbol | Meaning | Example |
|--------|---------|---------|
| `.` | Any character | `A.G` matches ACG, ATG, etc. |
| `^` | Start of line | `^ATG` matches ATG at start |
| `$` | End of line | `TAG$` matches TAG at end |
| `*` | Zero or more | `A*` matches "", "A", "AA", etc. |
| `+` | One or more (with -E) | `A+` matches "A", "AA", etc. |
| `[ABC]` | Character class | `[ATGC]` matches any nucleotide |
| `[^ABC]` | Negated class | `[^N]` matches any non-N |
| `{n}` | Exactly n (with -E) | `[ATGC]{3}` matches exactly 3 bases |

## Next Steps

You're now ready for Level 3, where we'll apply these regex skills specifically to FASTA file manipulation and learn more advanced pattern matching techniques for biological sequences!
