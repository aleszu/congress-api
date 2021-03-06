# How to use ProPublica's Congress API to see where senators stand on issues

Earlier this month, the *Los Angeles Times* used *ProPublica's* Congress API to [visualize the U.S. Senate's response](http://www.latimes.com/projects/la-na-pol-senators-trump-russia/) to President Donald Trump's firing of FBI director James Comey. Released to the public last month, *ProPublica's* Congress API – which was originally created by *The New York Times* – allows users to retrieve data on members, votes, bills, statements and other congressional goings-on.

The following is a tutorial uses the command line (Terminal on Mac) to query the [Congress API](https://projects.propublica.org/api-docs/congress-api/) for statements issued by senators on James Comey. Users must first request an API key from *ProPublica* [here](https://www.propublica.org/datastore/api/propublica-congress-api).

# Plan out your workflow

Before doing anything, it's a good idea to write out your workflow, i.e. what your question is, what data you're getting, where you're getting it, and what you intend to do with it. Be as specific as possible! Note: We're only *simulating* what the *L.A. Times* did, this is by no means the only way to query, explore and visualize data from the Congress API. 

* Using the command line, query the "[Congressional Statements by Search Term](https://projects.propublica.org/api-docs/congress-api/endpoints/#get-congressional-statements-by-search-term)" endpoint for mentions of "Comey."
* Save results into a JSON file.
* Convert JSON files into CSVs.
* Stack all CSVs into one master CSV.
* Open the master CSV, filter down by senators and by date, then click through to the URLs of each senator's statement on Comey. Rank these manually by adding a column titled "Gist" and using what the *L.A. Times* used: *Willing to let the investigation play out, Noncommittal, Expressed concerns, Calls for a special prosecutor or investigation, No public statement.*

# Make Congress API call through the command line 

Retrieve each page of 20 results individually from Congress API by supplying API key, the "comey" search term, and using the following:

`curl -H "X-API-Key: MY_KEY_HERE" "https://api.propublica.org/congress/v1/statements/search.json?query=comey&offset=0" > file.json`

To get back to before Comey's firing, we need to run this 16 times, each time incrementing the `offset` by 20 (until you reach 300) and renaming the output JSON file accordingly (file1.json, file2.json, etc.)

# Convert JSON files to CSVs using csvkit

Install csvkit on command line

`sudo pip install csvkit`

Then convert JSON files to csvs by providing csvkit's "in2csv" function with the key "results," which is where your data is nested: 

`in2csv --key results file2.json > file2.csv`

# Combine all the CSVs

Stack all csv files into one csv by dumping all files into a new folder, navigate to that folder in Terminal using `cd FOLDERNAME` and then run:

`cat file1.csv file2.csv file3.csv file4.csv file5.csv file6.csv ... file16.csv > file_combined.csv`

Notice we have headers every 20 rows. We could remove those manually in Excel. But why not do it programmatically?

## Remove headers from a CSV using the command line

To remove headers from file2.csv through file16.csv, dump all of those files (everything except file1.csv) into a directory, navigate to the directory in Terminal and then run:

`for filename in $(ls file*.csv); do sed 1d $filename >> final.csv; done`

Then move file1.csv into that same directory and stack it together with final.csv using:

`cat file1.csv final.csv > final_combined.csv`

# Open up your CSV

![](http://storybench.org/assets/comey-api-call.png)

# Look through the statements

Here, an example of a [statement](https://www.heitkamp.senate.gov/public/index.cfm/press-releases?ID=752050F4-BD1C-4D47-855C-78BAAD99CD7C) released on May 10 about Comey's firing:

![](http://storybench.org/assets/heitkamp-comey.png)

# A shallow dive into the data

Using a pivot table with our data, we reveal the spike of statements issued just after Trump fired Comey, which happened on the [afternoon](http://www.npr.org/2017/05/15/527773206/what-just-happened-the-james-comey-saga-in-timeline-form) of May 9.

### Statements about James Comey released by Congress in May 2017

![](http://storybench.org/assets/comey-chart.png)

# The pagination problem 

The Congress API only allows you to retrieve 20 results per query, which is why we had to change the offset value to "paginate" through their results. But the process of having to download 17 JSON files is a bit time-consuming (~ 1 minute each). Is there an easier way?

The following will retrieve the same results and save them all into one JSON file. 

```
for ((i=0;i<=300;i+=20)); 
	do curl -H "X-API-Key: MY_KEY_HERE" 
	"https://api.propublica.org/congress/v1/statements/search.json?query=MY_SEARCH_TERM_HERE&offset=$i"; 
done > file.json
```
Unfortunately, this method introduces some extraneous JSON that ProPublica adds: ("status":"OK", "copyright":"Copyright (c) 2017 Pro Publica Inc. All Rights Reserved.", etc.). If anyone has suggestions to improve this, please send along!





