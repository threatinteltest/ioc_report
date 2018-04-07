# [ioc_report]  
##### Generate a report containing IOCs gathered from VirusTotal and Hybrid-Analysis.  

#### Description  
This project is used as a tool to automate the process of gathering indicators of compromise (IOC) from VirusTotal or Hybrid-Analysis to sweep your environment. Since this project was built around public APIs, there is a rate limit. I used the VirusTotal rate limit (4 requests per minute at the time of writing this) as the baseline. Since the Hybrid-Analysis function of this script checks two environments (Win 7 32-bit and Win 7 64-bit), it takes approximately 45 seconds to pull the information for each URL provided when generating the full report. The basic report just provides the filetypes and checksums of the initial downloads so this should be 10x faster (FILE DOWNLOAD TIME x NUMBER OF URLS PROVIDED). If you have access to the private API, just remove the 15 second waits and everything "should" be fine (I haven't check the rate limits for the private API because I'm broke). Is 45 seconds a long time? Sure it is but this gives you more time to go do something else like flirt with your crush at work or twidle your thumbs.  

The `OUTPUTFILE` was meant to be used as a lookup table in Splunk in order to do more correlation with other log sources but as with any other open-source project, use it as you best see fit. If you are a Splunk master, manipulating the lookup table should be a piece of cake.  


#### Prerequisites  
- Python 2.7.14  
- Python Requests module  
- VirusTotal API key  
- Hybrid-Analysis API key and secret  

#### Setup  
Open a terminal and run the following commands:  
```bash
git clone https://github.com/leunammejii/ioc_report.git
cd ioc_report
```

#### Basic Report  
The basic report, `basic_report.sh`, is used to download the files, get the MIME-type, MD5, SHA256 hashes, remove the files, and write the comma-separated data to a file.  

To run the script, run the following command from the project directory:  
```bash
bash basic_report.sh INPUTFILE OUTPUTFILE
```

#### Full Report  
The full report, `full_report.sh`, is used to download the files, get the MIME-type, MD5, SHA256 hashes, remove the files, requests the checksums from VirusTotal for the last downloaded file it analyzed, request more information (filetypes, IP addresses, domains, and extracted file checksums) from Hybrid-Analysis, and write the data to a comma-separated or flat text file. There are two report types:  
- flat (simple text file list the IOCs by URL)  
- full (to be used as a lookup table in Splunk, auto-generated)  

You can request both at the same time if needed.  

To run the script, add the API keys for both VirusTotal and Hybrid-Analysis to `config.py`, add the full path of the input and output files to `config.py`, and run the following command from the project directory:  
```python
python full_report.py [--flat]
```

This will query VirusTotal for the hashes of the last downloaded the files from the URLs provided. If you want to download the files first to get the hashes, run the following command:  
```python
python full_report.py  [--flat] --download
```

#### Read the Results  
Both of these scripts will read a list of URLs from the `INPUTFILE` and write the data to the `OUTPUTFILE`. If you want to read the results of the CSV file from the commandline, run the following command:  
```bash
column -t -s , OUTPUTFILE
```

Or, you can just open the `OUTPUTFILE` in Excel (LibreOffice Calc). Sample outputs can be found [here](https://github.com/leunammejii/ioc_report/blob/master/sample_basic_results.csv) (basic report), [here](https://github.com/leunammejii/ioc_report/blob/master/sample_full_results.csv) (full report), and [here](https://github.com/leunammejii/ioc_report/blob/master/sample_flat_results.txt) (full report + flat report).  

#### Destroy
To remove the project completely,  run the following commands:  
```bash
rm -rf ioc_report
```  

#### Things to Know  
- If the MIME type of the `initial:filetype` does not start with `application/`, only use the IOCs with headers like `vt:` and `ha:` because the script submitted the SHA256 checksums from VirusTotal to Hybrid-Analysis.  
- If the MIME type of the `initial:filetype` starts with `application/` and you chose the option to download the file, know that any indicators found with VirusTotal are from the last file they downloaded from the site. Since you just downloaded the file, their indicators may be from an older file (but if the initial and vt checksums match, you're good to go). If this is the case, the script will submit the SHA256 checksums of the file you downloaded to Hybrid-Analysis.  
- The script uses two environments (Win 7 32-bit and 64-bit) in Hybrid-Analysis. There may be cases where the you get the message `file never seen` for one environment and not the other.  
- If the MIME type of the `initial:filetype` starts with `text/`, you may get a different hash each time if you select the option to download it since something within the HTML may have changed since the last download.  

#### To-Do  
- [ ] Remove duplicate lines in report  
