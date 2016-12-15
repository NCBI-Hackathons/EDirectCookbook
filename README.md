# EDirect_EUtils_API_Cookbook

## Usage

### Just copy and paste commands off the page.  Modify the search strings to work for you!

### If there are things you want to be able to do with EDirect, but can't figure out how, put in an issue using the Issues tab!

### To install EDirect, follow the instructions here:  https://www.ncbi.nlm.nih.gov/books/NBK179288/

## Posting
### The following items are reasonable to post on this page:
    Pull requests with working EDirect scripts
    Issues citing specific use cases and requesting EDirect scripts
    Issues presenting non-working EDirect scripts looking for fixes
    Pull requests with modifications or optimization of EDirect scripts on the page.
### Best Practices for EDirect:
    Please keep to <50,000 expected hits (it simply won’t work)
    Please do not run from multiple processors on a compute farm

### All items below come with no explicit or implicit warranty.  All code is as-is and produced for the bioinformatics community, from the bioinformatics community.  

## EDirect Scripts

### Gene Aliases

Description (optional):

Written by: NCBI Folks (12/6/2016)

Confirmed by:

Databases: gene

```
esearch -db gene -query "Liver cancer AND Homo sapiens" | \
efetch -format docsum | \
xtract -pattern DocumentSummary -element Name OtherAliases OtherDesignations
```

### Genomic sequence fastas from RefSeq assembly for specified taxonomic designation

Description (optional):

Written by: NCBI Folks (12/6/2016)

Confirmed by:

Databases: assembly

```
wget \`esearch -db assembly -query "Leptospira alstonii" | efetch -format docsum | xtract -pattern FtpPath -sep "\n" -element FtpPath | grep GCF | awk   -F"/" '{print $0"/"$NF"_genomic.fna.gz"}'\`
```

### Get organellar contigs from genbank

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: nuccore

```
esearch -db nuccore -query "LKAM01" | efetch -format fasta
```

### Get protein sequences from nucleotide accessions

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: nuccore, protein

```
cat accs_file | epost -db nuccore -format acc | elink -target protein | efetch -format fasta
```

### Complete taxonomy (KPCOFG) for taxids

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: taxonomy

```
efetch -db taxonomy -id 9606,1234,81726 -format xml | xtract -pattern Taxon -tab "," -first TaxId ScientificName -group Taxon -KING "(-)" -PHYL "(-)" -CLSS "(-)" -ORDR "(-)" -FMLY "(-)" -GNUS "(-)" -block "*/Taxon" -match "Rank:kingdom" -KING ScientificName -block "*/Taxon" -match "Rank:phylum" -PHYL ScientificName -block "*/Taxon" -match "Rank:class" -CLSS ScientificName -block "*/Taxon" -match "Rank:order" -ORDR ScientificName -block "*/Taxon" -match "Rank:family" -FMLY ScientificName -block "*/Taxon" -match "Rank:genus" -GNUS ScientificName -group Taxon -tab "," -element "&KING" "&PHYL" "&CLSS" "&ORDR" "&FMLY" "&GNUS"
```

### Obtain UniProt IDs from gene symbols

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: gene, protein

```
esearch -db gene -query "tp53[preferred symbol] AND human[organism]" | elink -target protein | esummary | xtract -pattern DocumentSummary -element Caption SourceDb | grep -E '^[OPQ][0-9][A-Z0-9]{3}[0-9]\|^[A-NR-Z][0-9]([A-Z][A-Z0-9]{2}[0-9]){1,2}'
```

### Retrieve Taxon IDs from list of genome accession numbers

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: nuccore

```
cat genome_accession.txt | epost -db nuccore -format acc | esummary | xtract -pattern DocumentSummary -element AccessionVersion TaxId
```

### Convert article DOI to PMID

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: pubmed

```
esearch -db pubmed -query "10.1111/j.1468-3083.2012.04708.x" | esummary | xtract -pattern DocumentSummary -block ArticleId -sep "\t" -tab "\n" -element IdType,Value | grep -E '^pubmed|doi'
```

### Access organism specific meta-data from NCBI genome database

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: genome, bioproject

```
esearch -db genome -query "22954[uid]" | elink -target bioproject | efetch -format xml | xtract -pattern DocumentSummary -element Salinity OxygenReq OptimumTemperature TemperatureRange Habitat
```

### Get the status of records from PubMed search

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: pubmed

```
esearch -db pubmed -query "pde3a AND 2016[dp]" | esummary | xtract -pattern DocumentSummary -element Id RecordStatus
```

### Sort the hits by sequence length in nucleotide database

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: nuccore

```
esearch -db nuccore -query "bacillus[orgn] AND biomol_rRNA[prop] AND 1500:1560[slen]" | esummary | xtract -pattern DocumentSummary -element Slen Extra | sort -rnk 1
```

### Getting meta data from assembly

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: assembly

```
esearch -db assembly -query "mammals[orgn] AND latest[filter]" | efetch -format docsum | xtract -pattern DocumentSummary -element Organism,SpeciesName,BioSampleAccn,LastMajorReleaseAccession -block Stat -if "@category" -equals chromosome_count -element Stat |grep -Pv "\t0$"
```

### Fetch HSPs from a BLAST hit in FASTA                                            	

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: nuccore

```
blastn -db nr -query in.fna -remote -outfmt "6 sacc sstart send" | xargs -n 3 sh -c 'efetch -db nuccore -id "$0" -seq_start "$1" -seq_stop "$2" -format fasta'
```

### Get all Gene Ontology IDs for a given protein accession

Description (optional):
Written by: NCBI Folks (12/6/2016)
Confirmed by:
Databases: protien, biosystems

```
epost -db protein -id BAD92651.1 -format acc | elink -target biosystems | efetch -format docsum | xtract -pattern externalid -element externalid | awk '{if ($0 ~ /GO/) print $0}'
```

### Get the ten most frequently-occurring authors for a set of articles
 
Description (optional): Searches PubMed for the string "traumatic brain injury athletes", restricts results to those published in 2015 and 2016, retrieves the full XML records for each of the search results, extracts the last name and initials of every author on every record, sorts the authors by frequency of occurrence in the results set, and presents the top ten most frequently-occurring authors, along with the number of times that author appeared.
Written by: Mike Davidson (NLM) (12/15/2016)
Confirmed by:
Databases: pubmed

```
esearch -db pubmed -query "traumatic brain injury athletes" -datetype PDAT -mindate 2015 -maxdate 2016 | efetch -format xml | xtract -pattern Author -sep " " -element LastName,Initials | sort-uniq-count-rank | head -n 10
```

### Look up the publication date for thousands of PMIDs 

Description (optional):

Written by: NCBI Folks (12/15/2016)

Confirmed by:

Databases: pubmed

```
cat table_of_pubmed_ids | epost -db pubmed | efetch -format xml | xtract -pattern PubmedArticle -element MedlineCitation/PMID -block PubDate -sep " " -element Year,Month MedlineDate
```
