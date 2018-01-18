# Complement 

## Generates a "complementary" .nessus v2 file from excel spreadsheet data

In an enterprise environment, the Nessus scanner does not check for all low-hanging fruit typically expected for even simple vulnerability assessment, for example default passwords are often missed. 
That means the scanner report need to be complemented with such new findings, to void the final recipient receiving multiple files with findings. 
That is what Complement is there for; add complementary vulnerabilities to an excel spreadsheet, compile a .nessus file with complement.py, and finally merge this file with your regular scanning file using the nessus butcher.py to create one single output package. 

## Example template 

First thing needed is to make an example spreadsheet as a template to use:
```
python3 complement.py --make-template example/complementary_findings.xlsx
```

![image of excel example template](/examples/excel.png) 


## Editing the excel spreadsheet
The first row is used as headers, dont change these! However you can re-arrange or add new headers. 
Each row is a finding. 
Findings with the same @pluginID may be grouped in the finial output, each with a unique IP:@port and plugin_output data. 
For example, in the example/complement_findings.xlsx ablove, note that @pluginID for row 2 and 3 are 999762, meaning findings can be grouped if needed.

Some special columns are needed:
- @pluginID should be a unique value. Nessus own IDs are 5-digits so pick a 6 digit random number not to colide. Use the same value if grouping is desired, complemented with a unique plugin_output
- risk_factor should be either of none, low, medium, high, or critical. The severity attribute is automatically calculated. 
- @pluginName is used as a title
- description means description of your finding
- solution means solution of your finding
- IP need to be one single IP address or hostname
- plugin_output is the unique data gathered from this hosts, then multiple findings are grouped with the same @pluginID 
- @protocol	is either tcp or udp
- @port	means tcp/udp port of the vulnerable remote service
- @svc_name is the remote service class, such as www,telnet,vnc, or x11. /svc_names.txt is a longer list of existing examples. 

## Compiling an .nessus file 

One or more excel spreadsheets can be added as input. Output is the .nessus v2 xml:
```
python3 complement.py example/complementary_findings.xlsx --output-file example/complementary_findings.nessus 
```

The result can be examined with for example the butcher:
```
python3 ../butcher/butcher.py example/complementary_findings.nessus --long
ID	severity	pluginName	IP
948095	Critical	SQL Injection	192.168.2.120
999762	High	Default administrative password	192.168.2.100
999762	High	Default administrative password	192.168.2.120
```

### Merge findings with scan results 
To merge findings you neen the butcher to compile new output:
```
python3 ../butcher/butcher.py ../butcher/examples/example_scan.nessus  example/complementary_findings.nessus --min-severity Critical 
ID	severity	pluginName	IP
948095	Critical	SQL Injection	192.168.2.120
63145	Critical	USN-1638-3 : firefox regressions	192.168.1.43
63023	Critical	USN-1636-1 : thunderbird vulnerabilities	192.168.1.43
...
```
Please note that the first finding is from the spreadsheet, and remaing from the butcher example scan. 

Here is a html example as well:
```
python3 ../butcher/butcher.py ../butcher/examples/example_scan.nessus example/complementary_findings.nessus --format html --output-file example/merged-findings.html
```	
![image of a merged HTML report](/examples/merged-findings.png) 


### Adding additional columns 
The first row is used as headers, where each column match a .nessus xml attribute or element. 
The most common attributes and elements are pre-populated in the template. 
Additional attributes or elemets can be added, however you need to understand the nessus_v2 format used, which is documented in the nessus_v2_file_format.pdf
Attributes starts with an @ sign and means attributes to the ReportItem element, such as pluginFamily or pluginID:
```
<ReportItem pluginFamily="Port scanners" pluginID="11219" pluginName="Nessus SYN scanner" port="80" protocol="tcp" severity="0" svc_name="www">
```
Everything not attributes are elements, such as synopsis:
``` 
<synopsis>The remote Windows host is affected by a remote code execution vulnerability.</synopsis>
``` 

Use for example the butcher's --format xml to inspect your output data and find out how nessus typically stores data:
``` 
python3 ../butcher/butcher.py ../butcher/examples/example_scan.nessus --id 63145 --format xml | head
<?xml version="1.0" ?><ReportItem pluginFamily="Ubuntu Local Security Checks" pluginID="63145" pluginName="USN-1638-3 : firefox regressions" port="0" protocol="tcp" severity="4" svc_name="general">	
	<cpe>cpe:/o:canonical:ubuntu_linux</cpe>	
	<cve>CVE-2012-4201</cve>	
	<cve>CVE-2012-4202</cve>	
```

## Prerequisites

The butcher is written in Python3. The following python3 packages are required:
- openpyxl

## License
Open Source MIT License




