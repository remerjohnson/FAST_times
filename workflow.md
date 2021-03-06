# General Workflow
+ Get e.g. `Topic.txt` (a list of all Subject:topics) from JIRA
+ Get a report on the encoding `file -bi Topic.txt`
+ Convert to UTF-8: ```iconv -f ISO-8859-15 -t UTF-8 Topic.txt > topic.txt```
+ Convert to csv: `cat topic.txt > topic.csv`
+ Cut columns we don't need via `csvkit`'s `csvcut`:  ```csvcut -t -c ark,topic topic.csv > new_topic.csv```
+ Convert to tsv: `cat new_topic.csv | sed 's/,/\t/g' > topic.tsv` or `cat new_topic.csv > topic.tsv`
+ Run script that creates all the csv files: `python less_FAST_times.py`
+ Run the concatenate script: `python concatenate.py`
+ Open result `concatenated.csv` in OpenRefine
+ Check for any obvious anomalies via faceting, clusters
+ Run reconciliation service for FAST by `cd` into the `fast-reconcile` repo and run: `python reconcile.py`
+ Select options on clean_labels column, then select reconcile: follow prompts
+ Review matches. When satisfied, extract label, URI, marc tag into new columns
+ Export to Excel for review and splitting off of unmatched terms
+ Repeat reconciliation for unmatched terms until all have been attempted
+ Once review of the Excel has occurred and sent back for more recon, get back into csv:
  + `in2csv topic.xlsx > topic.csv # Requires csvkit`
+ Repeat earlier steps for recon as necessary   

# From VIAF to Wikidata / Wikipedia using OpenRefine
[Help from the Refine GitHub Wiki](https://github.com/OpenRefine/OpenRefine/wiki/Fetching-URLs-From-Web-Services)

Some VIAF authorities have Wikidata URIs in them. Once we have those VIAF URIs, we can use Refine's "Create a column by fetching URLs" on this column.  

The data is in a JSON version of the authorities, which is appended to each URI as `/justlinks.json`  

Therefore, the expression, using GREL, will be `value + '/justlinks.json'`.  

We get back a bunch of JSON in this column, which we will need to parse. To parse out the Wikidata IDs, the GREL will be `grel:	value.parseJson()["WKP"]`    

## Extracting Wikipedia links from VIAF data

Using jython in OpenRefine's "Create column based on this column" on the VIAF JSON:
```
import re
g = re.search('.*(https:\/\/en\.wikipedia\.org\/wiki\/\w+).+', value)
return g.group(1)
```

