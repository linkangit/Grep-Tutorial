# Level 5: Advanced Regular Expressions

Now that you've mastered basic regex and file-specific applications, it's time to explore the most powerful grep features: grouping, backreferences, alternation, and complex pattern combinations.

## Learning Objectives

By the end of this level, you will be able to:
- Use grouping and capturing groups effectively
- Apply backreferences for pattern matching
- Master alternation for complex OR conditions
- Combine multiple regex elements into sophisticated patterns
- Optimize regex performance for large datasets

## Advanced Regex Concepts

### 1. Grouping and Capturing Groups

Groups allow you to treat multiple characters as a single unit and capture matched text for reuse.

#### Basic Grouping with Parentheses
```bash
# Group characters together
grep -E "(ATG)" sequences.fasta  # Simple grouping

# Group with quantifiers
grep -E "(ATG){2}" sequences.fasta  # Match ATGATG

# Group alternatives
grep -E "(start|begin|initiation)" annotations.gff
```

#### Capturing Groups for Complex Patterns
```bash
# Capture and reuse patterns (backreferences)
grep -E "([ATGC]{3})\1" sequences.fasta  # Find repeated triplets

# Match palindromic sequences
grep -E "([ATGC])([ATGC])\2\1" sequences.fasta  # ABBA pattern

# Complex gene ID patterns
grep -E "(ENSG[0-9]{11}).*\1" cross_references.txt  # Same gene ID appears twice
```

### 2. Backreferences

Backreferences allow you to match previously captured groups within the same pattern.

#### Basic Backreferences
```bash
# \1 refers to first captured group, \2 to second, etc.

# Find duplicated words
grep -E "([A-Za-z]+) \1" annotations.txt

# Find repeated nucleotide patterns
grep -E "([ATGC]{2,4})\1+" sequences.fasta  # Pattern repeated one or more times

# Palindromic restriction sites
grep -E "([ATGC])([ATGC])([ATGC])\3\2\1" sequences.fasta  # 6bp palindrome
```

#### Complex Backreference Examples
```bash
# Find genes with matching start and end annotations
grep -E "(gene_start:[0-9]+).*gene_end:\1" features.gff

# Match repeated motifs in proteins
grep -E "([A-Z]{3,5})[A-Z]*\1" proteins.fasta  # Motif appears twice

# Find tandem repeats
grep -E "([ATGC]{10,20})\1{2,}" sequences.fasta  # Pattern repeated 3+ times
```

### 3. Advanced Alternation

Go beyond simple OR conditions with complex alternation patterns.

#### Complex Alternation Patterns
```bash
# Multiple alternatives with different lengths
grep -E "(insulin|glucagon|somatostatin)" hormone_genes.fasta

# Mixed pattern types
grep -E "(chr[0-9]+|chrX|chrY|chrM)" coordinates.bed

# Functional categories
grep -E "(kinase|phosphatase|transferase|hydrolase)" enzymes.gff

# Organism variations
grep -E "(homo.sapiens|human|h\.sapiens)" sequences.fasta
```

#### Nested Alternation
```bash
# Complex nested patterns
grep -E "((start|begin|initiation).codon|(stop|termination).codon)" annotations.txt

# Multiple gene name formats
grep -E "(gene[_-]?(name|id|symbol)[:=]([A-Z0-9]+))" metadata.txt

# Version number patterns
grep -E "(v|version|ver)\.?[0-9]+(\.[0-9]+)*" software_versions.txt
```

### 4. Advanced Quantifiers

Precise control over pattern repetition.

#### Range Quantifiers
```bash
# Specific repeat counts
grep -E "[ATGC]{100,200}" sequences.fasta  # 100-200 nucleotides

# Minimum repeats
grep -E "A{10,}" sequences.fasta  # 10 or more A's

# Maximum repeats
grep -E "N{0,5}" sequences.fasta  # 0 to 5 N's

# Exact counts for biological patterns
grep -E "([ATGC]{3}){50,100}" sequences.fasta  # 50-100 codons
```

#### Non-Greedy Quantifiers (where supported)
```bash
# Note: grep doesn't support non-greedy quantifiers directly
# But we can achieve similar results with careful pattern construction

# Instead of .*? (non-greedy), use specific character classes
grep -E "ATG[^>]*?TAA" sequences.fasta  # May not work in all grep versions
```

### 5. Character Classes and Ranges

Advanced character matching techniques.

#### Custom Character Classes
```bash
# Purine bases (A, G)
grep -E "[AG]+" sequences.fasta

# Pyrimidine bases (C, T)
grep -E "[CT]+" sequences.fasta

# Hydrophobic amino acids
grep -E "[AILMFWYV]+" proteins.fasta

# Charged amino acids
grep -E "[DEKRH]+" proteins.fasta
```

#### Negated Character Classes
```bash
# Everything except standard nucleotides
grep -E "[^ATGCN]" sequences.fasta

# Non-digit characters in coordinates
grep -E "[^0-9]" positions.txt

# Find sequences without stop codons
grep -E "^ATG[^*]+$" translated_sequences.txt
```

## Complex Bioinformatics Applications

### Example 1: Gene Structure Validation
```bash
# Find genes with proper start-stop codon structure
grep -E "^ATG([ATGC]{3})*(TAA|TAG|TGA)$" cds_sequences.fasta

# Validate exon-intron boundaries (GT...AG)
grep -E "GT[ATGC]*AG" splice_sites.fasta

# Find genes with alternative start codons
grep -E "^(ATG|GTG|TTG)([ATGC]{3})*(TAA|TAG|TGA)$" alternative_starts.fasta
```

### Example 2: Protein Domain Analysis
```bash
# Find proteins with transmembrane domains (hydrophobic stretches)
grep -E "[AILMFWYV]{15,25}" proteins.fasta

# Identify signal peptides (hydrophobic start, cleavage site)
grep -E "^[AILMFWYV]{10,20}[AG][^P][ST]" proteins.fasta

# Find zinc finger domains
grep -E "C[A-Z]{2}C[A-Z]{12}H[A-Z]{3}H" proteins.fasta
```

### Example 3: Sequence Repeat Analysis
```bash
# Find simple tandem repeats
grep -E "([ATGC]{1,6})\1{5,}" sequences.fasta

# Identify microsatellites
grep -E "(CA){10,}|(GT){10,}|(ATCT){5,}" sequences.fasta

# Find inverted repeats (palindromes)
grep -E "([ATGC]{5,10})[ATGC]*([ATGC]{5,10})" sequences.fasta | grep -E "\2.*\1"  # Approximate
```

### Example 4: Genomic Coordinate Parsing
```bash
# Parse complex genomic coordinates
grep -E "(chr[0-9XYM]+):([0-9]+)-([0-9]+)(\([+-]\))?" coordinates.txt

# Extract specific chromosome ranges
grep -E "chr([1-9]|1[0-9]|2[0-2]):([0-9]{6,9})-([0-9]{6,9})" coordinates.txt

# Find overlapping regions (simplified)
grep -E "chr([0-9XY]+):[0-9]+-[0-9]+" regions.bed
```

## Advanced Pattern Examples

### Restriction Enzyme Sites
```bash
# EcoRI with ambiguous nucleotides
grep -E "G[AN]ATTC" sequences.fasta

# Multiple restriction sites
grep -E "(GAATTC|AAGCTT|GGATCC|CTGCAG)" sequences.fasta

# Palindromic restriction sites (general pattern)
grep -E "([ATGC])([ATGC])([ATGC])([ATGC])\4\3\2\1" sequences.fasta
```

### Gene Expression Analysis
```bash
# Find genes with expression level indicators
grep -E "(FPKM|TPM|RPKM):?[[:space:]]*([0-9]+\.?[0-9]*)" expression_data.txt

# Extract fold-change patterns
grep -E "fold.change:?[[:space:]]*([+-]?[0-9]+\.?[0-9]*)" diff_expression.txt

# P-value patterns
grep -E "p.val(ue)?:?[[:space:]]*([0-9]*\.?[0-9]+e?-?[0-9]*)" statistics.txt
```

### Phylogenetic Analysis
```bash
# Find branch length patterns in Newick format
grep -E ":[0-9]+\.[0-9]+" phylogenetic_trees.txt

# Extract species names from trees
grep -oE "[A-Z][a-z]+_[a-z]+" species_trees.txt

# Bootstrap support values
grep -E "\)([0-9]{2,3}):" phylogenetic_trees.txt
```

## Practice Exercises

### Exercise 1: Complex Gene Pattern Matching
Using `complex_sequences.fasta`:
1. Find genes that start with ATG and have exactly 3 exons (use backreferences)
2. Identify sequences with tandem repeats of 4-6 nucleotides
3. Find palindromic sequences of exactly 8 nucleotides
4. Extract genes with alternative polyadenylation signals

**Sample Solutions:**
```bash
# 1. Three exon genes (simplified pattern)
grep -E "^ATG([ATGC]*intron[ATGC]*){2}(TAA|TAG|TGA)$" sequences.fasta

# 2. Tandem repeats
grep -E "([ATGC]{4,6})\1+" sequences.fasta

# 3. Palindromes
grep -E "([ATGC])([ATGC])([ATGC])([ATGC])\4\3\2\1" sequences.fasta

# 4. Polyadenylation signals
grep -E "(AATAAA|ATTAAA|AGTAAA)" sequences.fasta
```

### Exercise 2: Protein Analysis
Using `protein_domains.fasta`:
1. Find proteins with repeated domains (same domain appears 2+ times)
2. Identify proteins with both kinase and phosphatase domains
3. Find proteins with conserved cysteine patterns (disulfide bonds)
4. Extract proteins with specific motif combinations

### Exercise 3: Genomic Annotations
Using `genome_features.gff`:
1. Find features that span multiple chromosomes (complex rearrangements)
2. Identify genes with overlapping coordinates on opposite strands
3. Extract features with specific attribute combinations
4. Find regulatory elements near gene boundaries

## Performance Optimization Tips

### Efficient Pattern Design
```bash
# Use specific character classes instead of .* when possible
grep -E "[ATGC]+" instead of grep -E ".*"

# Anchor patterns when appropriate
grep -E "^ATG" instead of grep -E "ATG"

# Use alternation efficiently
grep -E "(gene|mRNA|CDS)" instead of multiple grep commands
```

### Memory and Speed Considerations
```bash
# Use -F for literal strings when no regex needed
grep -F "GAATTC" sequences.fasta  # Faster than grep -E

# Combine patterns in single regex instead of multiple greps
grep -E "(pattern1|pattern2|pattern3)" file

# Use appropriate tools for very complex patterns
# Consider awk, sed, or perl for extremely complex operations
```

## Common Advanced Regex Pitfalls

1. **Catastrophic backtracking**: Avoid nested quantifiers like `(a+)+`
2. **Greedy quantifiers**: `.*` matches as much as possible
3. **Escaped characters**: Remember to escape special characters when literal
4. **Group numbering**: Groups are numbered left-to-right by opening parenthesis
5. **Performance**: Complex regex can be slow on large files

## Advanced Regex Debugging

```bash
# Test regex components separately
grep -E "([ATGC]{3})" sequences.fasta  # Test group
grep -E "\1" sequences.fasta  # Test backreference (won't work alone)

# Use -o to see exactly what matches
grep -oE "([ATGC]{2,4})\1+" sequences.fasta

# Add line numbers for debugging
grep -nE "complex_pattern" file.txt
```

## Integration with Other Tools

### Combining with awk for Complex Processing
```bash
# Use grep to filter, awk to process
grep -E "^>" sequences.fasta | awk -F'|' '{print $2}'

# Complex pattern matching with awk
awk '/^>/ && /insulin/ {flag=1; print} flag && /^[ATGC]/ {print; flag=0}' sequences.fasta
```

### Pipeline Integration
```bash
# Multi-step processing
grep -E "insulin" annotations.gff | \
grep -E "exon" | \
grep -oE "chr[0-9XY]+:[0-9]+-[0-9]+"
```

## Quick Reference - Advanced Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| `(pattern)` | Capturing group | `(ATG)` captures ATG |
| `\1` | Backreference to group 1 | `([ATGC]{3})\1` matches ATGATG |
| `(pat1\|pat2)` | Alternation | `(kinase\|phosphatase)` |
| `{n,m}` | Range quantifier | `[ATGC]{10,20}` |
| `[^ABC]` | Negated class | `[^N]` matches non-N |
| `(?:pattern)` | Non-capturing group | Not supported in grep |

## Next Steps

You're now ready for Level 6, where we'll apply these advanced regex skills to GFF/GTF file processing, learning how to parse complex genomic annotations and extract sophisticated biological information!

## Advanced Examples Summary

```bash
# Tandem repeats
grep -E "([ATGC]{2,10})\1{2,}" sequences.fasta

# Palindromic sequences
grep -E "([ATGC])([ATGC])([ATGC])\3\2\1" sequences.fasta

# Complex gene structure
grep -E "^ATG([ATGC]{3})*(TAA|TAG|TGA)$" cds.fasta

# Protein domains
grep -E "[AILMFWYV]{15,25}" transmembrane_proteins.fasta

# Genomic coordinates
grep -E "(chr[0-9XY]+):([0-9]+)-([0-9]+)" coordinates.bed
```
