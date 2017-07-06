# EDirect_EUtils_API_Cookbook


**Just copy and paste commands off the page.  Modify the search strings to work for you!**

**If there are things you want to be able to do with EDirect, but can't figure out how, you can ask the community for help by creating an Issue. See below, under "How to contribute," for more information.**

**To install EDirect, follow the instructions in ["Entrez Direct: E-utilities on the Unix Command Line"](https://www.ncbi.nlm.nih.gov/books/NBK179288/)**

## How to contribute

You can contribute to this page through GitHub. (If you are not already viewing the GitHub version of this page, please click the "View on GitHub" button at the top of the page.)  Using GitHub, you can create Issues or Pull Requests to contribute to the cookbook.

### Create an Issue to:

* Request an EDirect script to accomplish a task, citing specific use cases
* Present a non-working EDirect script and ask for a fix
* Identify non-working scripts listed below
	
### Create a Pull Request to:

* Add a working EDirect script to the list below
* Modify or optimize an EDirect script listed below
* Update the "Confirmed by:" date/version of a listed EDirect script with confirmation that it is still valid
	
## Best Practices for EDirect:

* Please keep to <50,000 expected hits (it simply won’t work)
* Please do not run from multiple processors on a compute farm

For more information and documentation on EDirect, please see:

* [Entrez Direct: E-utilities on the Unix Command Line](https://www.ncbi.nlm.nih.gov/books/NBK179288/)
* [Insiders Guide to Accessing NLM Data: EDirect Overview](https://dataguide.nlm.nih.gov/edirect/overview.html)

**All items below come with no explicit or implicit warranty.**

**All code is as-is and produced for the bioinformatics community, from the bioinformatics community.**

## EDirect Scripts


### Get all SRA runs for a given BioProject 
Description (optional):  
Written by: Bob Sanders (3/22/2017)  
Confirmed by:  
Databases: SRA, BioProject 

```
esearch -db bioproject -query "PRJNA356464" | elink -target sra | efetch -format docsum | \
xtract -pattern DocumentSummary -ACC @acc -block DocumentSummary -element "&ACC"
```

### Get latitiude and longitude for SRA Datasets (e.g. outbreaks and metagenomes)
Description (optional):  
Written by: BB, Mike D, Rob Edwards (4/12/2017)  
Confirmed by:  
Databases: SRA, BioSample 

```
for i in $(cat sra_ids.txt); do ll=$(esearch -db sra -query $i | \
elink -target biosample | efetch -format docsum | \
xtract -pattern DocumentSummary -block Attribute -if Attribute@attribute_name -equals lat_lon -element Attribute); \
echo -e "$i\t$ll"; done
```

### Get run sizes (in bp) for SRA Datasets
Description (optional): This retrieves the SRR id and the size in bp of the run from a file (`ids.txt`) of SRR IDs. You can also change `bases` to `size_MB`to get the size of the dataset in MB. _Question: Does the size in MB include the sequence identifiers (i.e. the size of the file) or just the sequences?_
Written by: Rob Edwards (7/6/2017)
Confirmed by:
Databases: SRA
```
epost -db sra -input ids.txt -format acc | esummary -format runinfo -mode xml | xtract -pattern Row -element Run,bases
```


### Gene Aliases
Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: gene  
```
esearch -db gene -query "Liver cancer AND Homo sapiens" | \
efetch -format docsum | \
xtract -pattern DocumentSummary -element Name OtherAliases OtherDesignations
```

### Genomic sequence fastas from RefSeq assembly for specified taxonomic designation
Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by: Peter Cooper (NCBI) and Wayne Matten (NCBI) (12/29/2016, v6.00)   
Databases: assembly  
```
wget `esearch -db assembly -query "Leptospira alstonii[ORGN] AND latest[SB]" | \
efetch -format docsum | \
xtract -pattern DocumentSummary -element FtpPath_RefSeq | \
awk -F"/" '{print $0"/"$NF"_genomic.fna.gz"}'`
```

```
(For larger sets of data the above may fail as wget may not accept a very large number of arguments.
The command below should work for all.)

esearch -db assembly -query "Leptospira alstonii[ORGN] AND latest[SB]" | \
efetch -format docsum | \
xtract -pattern DocumentSummary -element FtpPath_RefSeq | \
awk -F"/" '{print $0"/"$NF"_genomic.fna.gz"}' | \
xargs wget
```

### Get organellar contigs from genbank

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: nuccore  

```
esearch -db nuccore -query "LKAM01" | efetch -format fasta
```

### Get protein sequences from nucleotide accessions

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: nuccore, protein  

```
cat accs_file | epost -db nuccore -format acc | \
elink -target protein | efetch -format fasta
```

### Complete taxonomy (KPCOFG) for taxids

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: taxonomy  

```
efetch -db taxonomy -id 9606,1234,81726 -format xml | \
xtract -pattern Taxon -tab "," -first TaxId ScientificName \
-group Taxon -KING "(-)" -PHYL "(-)" -CLSS "(-)" -ORDR "(-)" -FMLY "(-)" -GNUS "(-)" \
-block "*/Taxon" -match "Rank:kingdom" -KING ScientificName \
-block "*/Taxon" -match "Rank:phylum" -PHYL ScientificName \
-block "*/Taxon" -match "Rank:class" -CLSS ScientificName \
-block "*/Taxon" -match "Rank:order" -ORDR ScientificName \
-block "*/Taxon" -match "Rank:family" -FMLY ScientificName \
-block "*/Taxon" -match "Rank:genus" -GNUS ScientificName \
-group Taxon -tab "," -element "&KING" "&PHYL" "&CLSS" "&ORDR" "&FMLY" "&GNUS"
```

### Obtain UniProt IDs from gene symbols

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: gene, protein  

```
esearch -db gene -query "tp53[preferred symbol] AND human[organism]" | \
elink -target protein | \
esummary | \
xtract -pattern DocumentSummary -element Caption SourceDb | \
grep -E '^[OPQ][0-9][A-Z0-9]{3}[0-9]\|^[A-NR-Z][0-9]([A-Z][A-Z0-9]{2}[0-9]){1,2}'
```

### Retrieve Taxon IDs from list of genome accession numbers

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: nuccore  

```
cat genome_accession.txt | \
epost -db nuccore -format acc | \
esummary | \
xtract -pattern DocumentSummary -element AccessionVersion TaxId
```

### Convert article DOI to PMID

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by: Mike Davidson (NLM) (12/16/2016, v5.80)  
Databases: pubmed  

```
esearch -db pubmed -query "10.1111/j.1468-3083.2012.04708.x" | \
esummary | \
xtract -pattern DocumentSummary -block ArticleId -sep "\t" -tab "\n" -element IdType,Value | \
grep -E '^pubmed|doi'
```

### Access organism specific meta-data from NCBI genome database

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: genome, bioproject  

```
esearch -db genome -query "22954[uid]" | \
elink -target bioproject | \
efetch -format xml | \
xtract -pattern DocumentSummary -element Salinity OxygenReq OptimumTemperature TemperatureRange Habitat
```

### Get the status of records from PubMed search

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by: Mike Davidson (NLM) (12/16/2016, v5.80)  
Databases: pubmed  

```
esearch -db pubmed -query "pde3a AND 2016[dp]" | \
esummary | \
xtract -pattern DocumentSummary -element Id RecordStatus
```

### Conduct a PubMed search and retrieve the results as a list of PMIDs

Description (optional):  
Written by: Mike Davidson (2/22/2017)  
Confirmed by: Mike Davidson (NLM) (2/22/2017, v6.30)  
Databases: pubmed  

```
esearch -db pubmed -query "seasonal affective disorder" | efetch -format uid
```

### Sort the hits by sequence length in nucleotide database

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: nuccore  

```
esearch -db nuccore -query "bacillus[orgn] AND biomol_rRNA[prop] AND 1500:1560[slen]" | \
esummary | \
xtract -pattern DocumentSummary -element Slen Extra | \
sort -rnk 1
```

### Getting meta data from assembly

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: assembly  

```
esearch -db assembly -query "mammals[orgn] AND latest[filter]" | \
efetch -format docsum | \
xtract -pattern DocumentSummary -element Organism,SpeciesName,BioSampleAccn,LastMajorReleaseAccession \
-block Stat -if "@category" -equals chromosome_count -element Stat | \
grep -Pv "\t0$"
```

### Fetch HSPs from a BLAST hit in FASTA                                            	

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: nuccore  

```
blastn -db nr -query in.fna -remote -outfmt "6 sacc sstart send" | \
xargs -n 3 sh -c 'efetch -db nuccore -id "$0" -seq_start "$1" -seq_stop "$2" -format fasta'
```

### Get all Gene Ontology IDs for a given protein accession

Description (optional):  
Written by: NCBI Folks (12/14/2016)  
Confirmed by:  
Databases: protien, biosystems  

```
epost -db protein -id BAD92651.1 -format acc | \
elink -target biosystems | \
efetch -format docsum | \
xtract -pattern externalid -element externalid | \
awk '{if ($0 ~ /GO/) print $0}'
```

### Get the ten most frequently-occurring authors for a set of articles
 
Description (optional): Searches PubMed for the string "traumatic brain injury athletes", restricts results to those published in 2015 and 2016, retrieves the full XML records for each of the search results, extracts the last name and initials of every author on every record, sorts the authors by frequency of occurrence in the results set, and presents the top ten most frequently-occurring authors, along with the number of times that author appeared.  
Written by: Mike Davidson (NLM) (12/15/2016)  
Confirmed by: Mike Davidson (NLM) (12/16/2016)  
Databases: pubmed  

```
esearch -db pubmed -query "traumatic brain injury athletes" -datetype PDAT -mindate 2015 -maxdate 2016 | \
efetch -format xml | \
xtract -pattern Author -sep " " -element LastName,Initials | \
sort-uniq-count-rank | \
head -n 10
```

### Get the ten funding agencies who are most active in funding articles on a particular topic

Description (optional): Searches PubMed for the string "diabetes AND pregnancy", restricts results to those published in 2014 through 2016, retrieves the full XML records for each of the search results, extracts the funding agencies for every  grant on every record, sorts the agencies by frequency of occurrence in the results set, and presents the top ten most frequently-occurring agencies, along with the number of times that agency appeared.  
Written by: Mike Davidson (2/17/2017)  
Confirmed by:  Mike Davidson (NLM) (v6.30, 2/17/2017)  
Databases: pubmed  

```
esearch -db pubmed -query "diabetes AND pregnancy" -datetype PDAT -mindate 2014 -maxdate 2016 | \
efetch -format xml | \
xtract -pattern Grant -element Agency | \
sort-uniq-count-rank | \
head -n 10
```

### Look up the publication date for thousands of PMIDs (option one)

Description (optional):  Takes a file which contains a list of PMIDs (table_of_pubmed_ids) and uses `cat` to access the contents of the file, `epost` to post the PMIDs to the history server, `efetch` to retrieve the records and `xtract` to extract PMID and Publication Date.  
Written by: NCBI Folks (12/15/2016)  
Confirmed by:  Mike Davidson (NLM) (v6.30, 2/17/2017)  
Databases: pubmed  

```
cat table_of_pubmed_ids | \
epost -db pubmed | \
efetch -format xml | \
xtract -pattern PubmedArticle -element MedlineCitation/PMID \
-block PubDate -sep " " -element Year,Month MedlineDate
```

### Look up the publication date for thousands of PMIDs (option two)

Description (optional): Takes a file which contains a list of PMIDs (table_of_pubmed_ids) and `epost -input` to access the contents of the file and post the PMIDs to the history server, `efetch` to retrieve the records and `xtract` to extract PMID and Publication Date.  
Written by: Mike Davidson (2/17/2017)  
Confirmed by:  Mike Davidson (NLM) (v6.30, 2/17/2017)  
Databases: pubmed  

```
epost -input table_of_pubmed_ids -db pubmed | \
efetch -format xml | \
xtract -pattern PubmedArticle -element MedlineCitation/PMID \
-block PubDate -sep " " -element Year,Month MedlineDate
```

### Find the first author for a set of PubMed records

Description (optional): Outputs the PMID and first author's last name and initials for one or more PubMed records
Written by: Mike Davidson (2/17/2017)  
Confirmed by:  Mike Davidson (NLM) (v6.30, 2/17/2017)  
Databases: pubmed  

```
efetch -db pubmed -id 16940437 -format xml | \
xtract -pattern PubmedArticle -element MedlineCitation/PMID \
-block Author -position first -sep " " -element LastName,Initials
```


### Download GEO Data from a BioProject Accession 

Description (optional):  
Written by: NCBI Folks (12/16/2016)  
Confirmed by:  
Databases: gds  

```
esearch -db gds -query "PRJNA313294[ACCN]" | \
efetch -format docsum | \
xtract -pattern DocumentSummary -element FTPLink
```

### Look up the publication date for thousands of PMIDs (option two)

Description (optional): Takes a file which contains a list of PMIDs (table_of_pubmed_ids) and `epost -input` to access the contents of the file and post the PMIDs to the history server, `efetch` to retrieve the records and `xtract` to extract PMID and Publication Date.  
Written by: Mike Davidson (2/17/2017)  
Confirmed by:  Mike Davidson (NLM) (v6.30, 2/17/2017)  
Databases: pubmed  

```
epost -input table_of_pubmed_ids -db pubmed | \
efetch -format xml | \
xtract -pattern PubmedArticle -element MedlineCitation/PMID \
-block PubDate -sep " " -element Year,Month MedlineDate
```
