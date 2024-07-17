# FuncFetch
<img src = "https://github.com/moghelab/funcfetch/blob/main/images/goldenRetriever.jpg" width="190" height="150" align="right" alt="Imaged" > 

The development of the FuncFetch workflow was funded by the National Science Foundation IOS-Plant Genome Research Program [Award #2310395](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2310395&HistoricalAwards=false). Additional project details can be found on the [Moghe Lab Tools](https://tools.moghelab.org/funczymedb) website. This Repository is meant to version-control and share the scripts and the workflow. The outputs of this workflow can be found on the Moghe Lab Tools website as well as the [FuncZymeDB repository](https://github.com/moghelab/funczymedb) 







# FuncFetch workflow scripts

## Step 1: Retrieve paper titles and abstracts from PubMed
Edit the Step 1 config file. You can find the template config file in the config folder in this repository.
```
[entrez]
email = ENTER_YOUR_EMAIL
api_key = YOUR_PUBMED_API_KEY
requests_per_second = 200
tool = biopython

[query_settings]
query = YOUR_ENZYME_FAMILY_QUERY

[journal_list]
journallist = main_journalsList.tab
reviewfilter = yes

[keywords_list]
keywords_list = main_keywordsList.tab

[use_elink]
use_elink = no
```

Run the script
```console
python funcfetch_step1.py --config bahd_step1_efetch.config
```
This will retrieve the following files to your working directory
```console
BAHD_acyltransferase_OR_hydroxycinnamoyl-step1-ncbiAbstracts.txt
BAHD_acyltransferase_OR_hydroxycinnamoyl-step1-ncbiQuery.txt
BAHD_acyltransferase_OR_hydroxycinnamoyl-step1-ncbiSummary-failedScreen.txt
BAHD_acyltransferase_OR_hydroxycinnamoyl-step1-ncbiSummary.txt
```
The .log file will note the parameters used and the output
```
Your selections are as follows:
CONFIG_FILE: bahd_step1_efetch.config
EMAIL: ....
APIKEY: ....
TOOL: biopython
QUERY: BAHD acyltransferase OR hydroxycinnamoyl
JOURNALS_LIST: main_journalList.tab
FILTER REVIEWS: yes
KEYWORDS_LIST: main_keywordsList.tab.fil
USE_ELINK: no

Output stats:
# of hits to Initial query: 667
Total # of hits to Initial query: 667
# of abstracts in Initial after all filters applied: 461
```
Some of these files will be passed on to Step 2

## Step 2: Use LLM to screen abstracts

If you don't already have an account with OpenAI, make one.
https://platform.openai.com/signup

Navigate to the OpenAI API platform dashboard and generate a secret key under "API Keys" if you don't already have one. Per OpenAI's recommendations: "Do not share your API key with others, or expose it in the browser or other client-side code." This API key will be required to run Step 2 and Step 4, so record it in a secure location.

Edit the Step 2 config file. You can find the template config file in the config folder in this repository.
```
[openai]
key = YOUR_OPENAI_API_KEY
# The organization field is optional and only necessary if your openai user account belongs to multiple organizations.
organization = 

[rate_limit]
# Request limits may vary depending on the OpenAI account and policies. For more information on limits, look under "Organization" "Limits" in the settings section of the OpenAI API platform.
requests_per_minute = 5000

[model_settings]
# We tested the FuncFetch workflow with the gpt-4-turbo-2024-04-09 model, but feel free to swap in a more recent model.
model = gpt-4-turbo-2024-04-09
temperature = 0.5

[query_settings]
# We recommend using the same query for all workflow steps, as this replicates the testing conditions.
query = YOUR_ENZYME_FAMILY_QUERY

[questions]
question_11 = Is this likely a paper that describes the biochemical (i.e. catalytic) activity of a {query} enzyme? Ideally, this confirmation would include an in vitro biochemical assay. Answer only with a 1 or 0, corresponding to yes or no respectively.
question_10 = Is this likely a paper that demonstrates the biochemical (i.e. catalytic) activity of a {query} enzyme? Ideally, this confirmation would include an in vitro biochemical assay. Answer only with a 1 or 0, corresponding to yes or no respectively.
question_9 = Is this likely a paper that directly tests the biochemical (i.e. catalytic) activity of a {query} enzyme? Ideally, this confirmation would include an in vitro biochemical assay. Answer only with a 1 or 0, corresponding to yes or no respectively.
question_6 = Is this abstract likely to describe a paper that directly tests the biochemical (i.e. catalytic) activity of a {query} enzyme? Ideally, this confirmation would include an in vitro biochemical assay. Answer only with a 1 or 0, corresponding to yes or no respectively.
question_3 = Is this abstract likely to describe a paper that conducted a direct assay of a {query} enzyme catalyzing a reaction with a chemical substrate? Answer only with a 1 or 0, corresponding to yes or no respectively.
```

Run the script
```console
python funcfetch_step2.py --config funcfetch_step2.config --abstracts ENZYME_FAMILY-step1-ncbiAbstracts.txt --summary ENZYME_FAMILY-step1-ncbiSummary.txt

# For details on arguments and parameters run this line:
python funcfetch_step2.py -h
```

This step will produce some JSONL files that are the input and output for the OpenAI batch API run. They will look like the following:
* ENZYME_FAMILY_batch_input.jsonl
* ENZYME_FAMILY_batch_output.jsonl
Additionally, the script with produce human and machine readable files detailing the filtered papers and their score as relevant or irrelevant:
* ENZYME_FAMILY_filtered_abstracts.tsv
* ENZYME_FAMILY_filtered_abstracts.txt
The RIS format file is the most important output and required as input into Zotero for Step 3:
* ENZYME_FAMILY_relevant_papers.ris

## Step 3: Get papers using Zotero
* Create a new collection in Zotero
* Load the RIS file generated by Step 2 into the collection
* Once the library is created, select all articles, right click and select "Find PDFs..."
* Most of the papers will be downloaded automatically
* For the remaining papers, click on the DOIs, go to the page, download the PDF, drag the PDF into the Zotero entry manually.
* Once all PDFs are obtained, export the collection as a csv file (the default)
* Navigate to the advanced settings section of Zotero
* Note the "Data Directory Location" under the Files and Folders tab
  
## Step 4: Use LLM to extract activity and metadata
## Step 5: Filter
```console
python funcfetch_step5.py species.taxonomy BAHD_acyltransferase_or_hydroxycinnamoyl_transferase_merge_method.flag.tsv
Reading Species info...
Parsing FuncFetch output file...
>>>Missing: Saccharothrix_espanensis --> Taxonomy added as NA
>>>Missing: Saccharothrix_espanensis --> Taxonomy added as NA
>>>Missing: Dendranthema_× --> Taxonomy added as NA
>>>Missing: Dendranthema_× --> Taxonomy added as NA
>>>Missing: Apple_tree --> Taxonomy added as NA
>>>Missing: Eggplant_(Solanum --> Taxonomy added as NA
>>>Missing: Chicory_(Cichorium --> Taxonomy added as NA
>>>Missing: Dendranthema_× --> Taxonomy added as NA
>>>Missing: Dendranthema_× --> Taxonomy added as NA
>>>Missing: Dendranthema_× --> Taxonomy added as NA
>>>Missing: Dendranthema_× --> Taxonomy added as NA
>>>Missing: Dendranthema_× --> Taxonomy added as NA
>>>Missing: Physcomitrella_patens --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Lycopersicon_esculentum --> Taxonomy added as NA
>>>Missing: Physcomitrella_patens --> Taxonomy added as NA
>>>Missing: Physcomitrella_patens --> Taxonomy added as NA
>>>Missing: Physcomitrella_patens --> Taxonomy added as NA
# of species with activity info: 1527
# of plant species with activity info: 1455
# of bacterial species with activity info: 15
# of other species with activity info: 57
Done!
```
This script will create two files -- a .log file and a .tax file. The .tax file will have the taxonomic associations in a tabular format
```
TITLE   DOI     SPECIES Family  Kingdom ENZYME_COMMON_NAME      ENZYME_FULL_NAME        GENBANK UNIPROT_ID      ALT_ID  SUBSTRATE       PRODUCT FLAG
Analysing a Group of Homologous BAHD Enzymes Provides Insights into the Evolutionary Transition of Rosmarinic Acid Synthases from Hydroxycinnamoyl-CoA:Shikimate/Quinate Hydroxycinnamoyl Transferases.    10.3390/plants13040512  Mentha longifolia       4136|Lamiaceae  33090|Viridiplantae     MlAT1   Hydroxycinnamoyl-CoA:Shikimate/Quinate Hydroxycinnamoyl Transferase     NA      NA      NA         p-coumaroyl-CoA; shikimate      p-coumaroyl shikimate   pass
```
This file can be opened in Excel to manually curate the rest of the way. This curation can be done in a stepwise manner. You may see the Excel files deposited in the Minimally Curated Sets folder to see how we performed this curation.

## Questions/concerns
If you have any questions or concerns, or just want to thank us, email Nathaniel Smith (nss97) or Gaurav Moghe (gdm67). We have Cornell email addresses. 
