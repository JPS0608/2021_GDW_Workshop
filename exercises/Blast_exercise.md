# Command line BLAST
In this exercise, we will learn how to use the command line blast tool, also known as BLAST+.  We will use these tools to:
- blast search NCBI databases remotely
- build our own blast database
- blast locally
- retrieve information from blast hits

The BLAST+ suite comes with the following tools.  We won't use all of them, but feel free to read about them [here](https://www.ncbi.nlm.nih.gov/books/NBK279690/):  You should recognize some of them from the lecture earlier.
- blast_formatter
- blastn
- deltablast
- makembindex
- rpstblastn
- update_blastdb.pl
- blastdb_aliastool
- blastp
- dustmasker
- makeprofiledb
- segmasker
- windowmasker
- blastdbcheck
- blastx
- legacy_blast.pl
- psiblast
- tblastn
- blastdbcmd
- convert2blastmask
- makeblastdb
- rpsblast
- tblastx

We will be using the 'terminal' on the mac. Please don't hesitate to ask if you have any questions regarding commands, parameters, etc.  
Open up the terminal as you learned previously.  
Here is a quick reminder of some basic commands (no need to enter them now).
```bash
# How to get the manual or help menu for a command/program ("grep" example)
man grep
grep -h

# change into a folder
cd Folder

# List the contents of a folder
ls
ls -lh
ls -lh Folder

# Open a file to the screen
cat file

# Open large files page by page, without wrapping lines
less -S file
   # type "q" to quit
   
# Write result of command to a new file
cat file > newfile

# Look at the first 10 lines of a file
head -10 file

# Last 10 lines
tail -10 file

# Search for lines matching "abc" in a file
grep "abc" file

# Select a column from a delimited ("tab" by default)
cut -f3 file
```
Notice that there are lines above that begin with `#`.  These are called "comments", and are ignored by the command line, so you can copy and paste them along with any commands.  

## Part 1:  Remote BLAST
When we blast "remotely", we are using our command line to submit a blast search to the NCBI server.
This means we must have an internet connection in order to perform the search.

**Note**: The time to complete a remote blast will vary depending on the sequence length, complexity, number of sequences, database, and internet speed.
For this first example, let's download an example protein sequence:
Wild camel (*Camelus ferus*) ferritin light chain protein [EPY89138.1](https://www.ncbi.nlm.nih.gov/protein/EPY89138.1).

Select -> Send To -> File -> Format: fasta  
This should be saved in the `Downloads` (`/Users/gdw/Downloads`) folder as "sequence.fasta"  
Now let's BLAST!!!
```bash
# First, let's make sure we are starting from the Desktop
cd
cd Desktop

# Make a new folder and move into it
mkdir BLAST_PRACTICE
cd BLAST_PRACTICE

# Lets move the camel sequence file here and rename it to something more useful
mv ~/Downloads/sequence.fasta camel_ferritin.faa

# Open the contents of the file
cat camel_ferritin.faa

# To get the help menu for any of our blast tools, type:
blastp -help

# Now Blast to the refseq-protein database
blastp \
   -query camel_ferritin.faa \
   -db swissprot \
   -remote \
   -out camel_ferritin.blastout

# Open the file to the screen (feel free to also try the commands `more`, `less`, `head`, or `tail` to read the file)
cat camel_ferritin.blastout

# Repeat the above search, but limit it to a specific species, e.g. alpacas
blastp \
   -query camel_ferritin.faa \
   -db swissprot \
   -remote \
   -out alpaca_ferritins.blastout \
   -entrez_query "Alpaca[ORGN]"

```
The `\` at the end of the line tells the computer that the command will continue onto the next line. This notation can help make really long commands look cleaner and easier to understand. For example, the commands  
```
head camel_ferritin.blastout
```
and
```
head \
   camel_ferritin.blastout
```
are equivalent.
This search should take a couple minutes at most.  Feel free to try and modify the above command to search your favorite taxonomic group. Open the contents of the files and explore:
```bash
# Open the second blast result file to the screen
cat alpaca_ferritins.blastout

# More convenient way of opening large files:
less -S alpaca_ferritins.blastout

# NOTE: simply type the letter 'q' to exit the `less` viewer 
```
The results look good. How many matches did you find in each file (hint, try counting lines using the command `wc -l`)?
However, this output format can be difficult to parse if we have thousands and thousands of sequences.
Let's repeat the search again, but this time with some changes.
```bash
# New Blast Search
blastp \
   -query camel_ferritin.faa \
   -db swissprot \
   -remote \
   -num_alignments 10 \
   -outfmt 7 \
   -out camel_ferritin.blastout.tsv
```
Open the contents of the new output file.
What is different?  What does the 'num_alignments' parameter mean? Do you think this would be easier or more difficult than the previous output to calculate various statistics, make plots, etc?  The result is actually a tab-delimited file, which can easily be imported into Excel or other spreadsheet software.  What does each column represent (hint: try looking at the manual using `blastp -help`)?

## Part 2:  Building a database and local BLAST
The remote blast above is convenient, but when no internet connection is available, or when there are thousands of sequences, this may not be optimal.  In these cases and many others, it is easiest to build your own BLAST database and perform the search on your own computer.

In this section, we are going to download the transcriptome (remember what a 'transcriptome' is?) from the pathogen *Trichinella patagoniensis*.  [Krivokapich et al, 2012 doi:10.1016/j.ijpara.2012.07.009](http://www.sciencedirect.com/science/article/pii/S0020751912001932). This genus of nematode worms causes the disease trichinellosis, and infect and/or are transmitted between a variety of mammals, including humans. This particular species was described from South American pumas. We will download the transcriptome from NCBI's Transcriptome Shotgun Assembly ([TSA](https://www.ncbi.nlm.nih.gov/genbank/tsa/)) database.
From this [TSA](https://www.ncbi.nlm.nih.gov/genbank/tsa/) link (I recommend to "control+click" on the link then select "Open Link in New Tab":
- Search for accession GECA00000000.1
- Click on contig link at bottom (To the right of "TSA"), it looks like [GECA01000001-GECA01039180](https://www.ncbi.nlm.nih.gov/Traces/wgs?val=GECA01)
- Go to download tab
- Click and download fasta link (GECA01.1.fsa\_nt.gz)
- Move the downloaded file into your current directory
```bash
mv /Users/gdw/Downloads/GECA01.1.fsa_nt.gz .
```

Alternatively, you can download directly from the command line using the command below:
```bash
curl -O ftp://ftp.ncbi.nlm.nih.gov/sra/wgs_aux/GE/CA/GECA01/GECA01.1.fsa_nt.gz
```
Easy, huh!  
Now let's process the data and build a blastable database
```bash
# Uncompress the file.  What format is it?
gunzip GECA01.1.fsa_nt.gz

# How many sequences are there?
grep -c "^>" GECA01.1.fsa_nt

# Get help menu for building a database
makeblastdb -help

# Build the database
makeblastdb \
   -in GECA01.1.fsa_nt \
   -input_type fasta \
   -dbtype nucl \
   -title Tpat \
   -parse_seqids \
   -out Tpat
```
What do the output files look like?  Can you open them?

Let's select some random sequences from the transcriptome to use as a query.  We will use the [seqtk](https://github.com/lh3/seqtk) toolkit from Heng Li. This toolkit is fast and a standard for basic processing of sequence files (fasta and fastq).
```bash
# Get a list of the subprograms in 'seqtk'
seqtk

# Get the manual for a particular sub-program of 'seqtk'
seqtk sample
seqtk seq

# Select 5 sequences at random
seqtk sample GECA01.1.fsa_nt 5 > sample5.fasta

# Remember how to check the number of sequences?
grep -c "^>" sample5.fasta
```
Next, we are going to blast these 5 sequences against the transcriptome database we made. What do you expect to find?
```bash
blastn \
   -query sample5.fasta \
   -db Tpat \
   -out sample5.blastout \
   -outfmt 6
```
How many hits are there in total?  Notice that now the output format is type "6" and not "7".  What is different between these formats (hint: check `blastn -help`)?
Let's repeat but filter for only the extremely small e-values.
Do you expect more matches or fewer matches?
```bash
blastn \
   -query sample5.fasta \
   -db Tpat \
   -out sample5.blastout2 \
   -outfmt 6 \
   -evalue 1e-50
```
For the last part, we are going to retrieve the sequences for our matches from the above BLAST search.
To do this, we need to make a list of the accession numbers of our matches (This only works if we built the database using the 'parse_seqids' parameter), and then retrieve the matching sequences.
```bash
# Grab the second column from the blast results
cut -f2 sample5.blastout2 > matches.list

# Retrieve the sequences for these matches
blastdbcmd \
   -db ./Tpat \
   -entry_batch matches.list \
   -out matches.fasta

# Interested in other formats or options?
blastdbcmd -help
```

## Part 3: Extra credit
[Wu et al. 2008 (doi:10.1016/j.parint.2008.03.005)](http://www.sciencedirect.com/science/article/pii/S1383576908000391) identified a set of candidate genes that were differentially expressed between various *Trichinella* infections.  One of these genes was the pax7 protein, accession: [KRY39300.1](https://www.ncbi.nlm.nih.gov/protein/954373245?report=fasta).
Is this gene expressed (i.e. found) in the *Trichinella patagoniensis* transcriptome?
If so, are there multiple paralogs?  
Hint: you will be searching a nucleotide database with a protein sequence.




