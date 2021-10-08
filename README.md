# dhcpcsv2pfsense - Merge DHCP static assignments .csv data into pfSense DHCP Server config file

Manage your DHCP static assignments in Excel (or a text editor) and easily transfer them into pfSense.

` `  
## Usage
```
$ ./dhcpcsv2pfsense -h
usage: dhcpcsv2pfsense [-h] [-v] [-V] Config_backup CSV_master_list

Merge DHCP static assignments .csv data into pfSense DHCP Server config file.
V0.0 211007

positional arguments:
  Config_backup    Path to dhcpd-config...xml backup.  If to dir then use most recent file.  If to file then use that specific file.
  CSV_master_list  Path to CSV master list file

optional arguments:
  -h, --help       show this help message and exit
  -v, --verbose    Print status and activity messages.
  -V, --version    Return version number and exit.

```

` `  
## Example run
```
$ ./dhcpcsv2pfsense /mnt/Downloads dhcp_master_list.csv --verbose
Merge DHCP static assignments .csv data into pfSense DHCP Server config file.
V0.0 211007
Using config backup input file    </mnt/Downloads/dhcpd-config-pfSense.lan-20211006220611.xml>
Using config backup output file   </mnt/Downloads/dhcpd-config-pfSense.lan-20211006220611.xml-csv1>
Using CSV master list input file  <dhcp_master_list.csv>
Processed 22 DHCP static assignments.
```

` `  
## Setup and Usage notes
- Supported on Python3.6+ only.  Developed on Centos 7.9 with Python 3.6.8.  Also tested on Windows 10.
- Tested on pfSense+ version 21.05.1-RELEASE.
- Place the files in a directory on your computer.
- Edit your master list .csv file to your needs.  Save from Excel as "CSV (MS-DOS)" format, not "CSV UTF-8".
` `
- To update/import the DHCP static assignments defined in your .csv file into pfSense:
  1) Diagnostics > Backup & Restore
     - Backup area > "DHCP Server", then "Download configuration as XML"
  2) Run this script
     - The first argument can be a path to a directory or to a specific file.  If to a directory then the newest dhcp-config...xml file is used.
     - The generated output .xml file has the same name as in the input file plus "-csv1" appended.  The numeric part increments as needed.
  3) Diagnostics > Backup & Restore
     - Restore area:  "DHCP Server", select the new file, "Restore Configuration"
  4) Services > DHCP Server
     - Confirm your DHCP Static Mappings look correct
     - "Apply Changes" (may take a few seconds)
     - If Apply Changes button does not show up at the top, then you may hit the Restart Service (circle arrow top right, may take a few seconds) for good measure, but is seems the updated data is already being used.
  5) Services > DNS Resolver
     - Restart Service (circle arrow top right, may take a few seconds)
     - Only needed if "Static DHCP - Register DHCP static mappings in the DNS Resolver" is enabled.

` `  
## Additional Usage Notes
- dhcpcsv2pfsense builds \<staticmap> bocks in the generated .xml file, one for each line in the .csv file.
- At least one DHCP Static Mapping assignment within pfSense is needed so that the exported config backup has a \<staticmap> block for the script to pick up as a  template.  Create at least one static assignment before generating the backup file.
The first existing \<staticmap> block is used as the template.  
- Field/column names in the .csv that match fields in the \<staticmap> template will overwrite the default values provided from the template.
- You may add columns to the .csv to set values in other fields in the \<staticmap> structure, as needed.
- The order of columns in the .csv file is not important.
- Fields in the .csv that have no match in the \<staticmap> are ignored, with verbose mode warnings.  If the field names change in the pfSense backup and are inconsistent with your .csv file, the script will warn you (with the `--verbose` switch).
- Fields in the \<staticmap> with no matching column in the .csv are kept as in the template - usually blank / empty, with one known exception:  "ddnsdomainkeyalgorithm = hmac-md5".
- Blank lines are permitted in the .csv file, with no output to the generated config file.  _A .csv line is taken as blank if the first column value is blank._


` `  
## Known issues:
- No error checks are performed on the correctness of your .csv file, such as valid IP or MAC addresses, or illegal characters in hostname, for example.  Missing or invalid cell values are taken verbatim.

` `  
## Version history
- V0.0 211007  New