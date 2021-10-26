# dhcpcsv2pfsense - Merge DHCP static assignments .csv data into pfSense DHCP Server config file

Manage your DHCP static assignments in Excel (or a text editor) and easily transfer them into pfSense.

dhcpcsv2pfsense V1.0 manages DHCP static assignments for multiple VLANs, all in one master list file.

` `  
## Usage
```
$ ./dhcpload -h
usage: dhcpload [-h] [-v] [-V] Config_backup CSV_master_list

Merge DHCP static assignments .csv data into pfSense DHCP Server config file. 
V1.0 211014

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
$ ./dhcpcsv2pfsense /path/to/folder/containing/dhcpd_backup/ /path/to/DHCP_master_list.csv -v
Merge DHCP static assignments .csv data into pfSense DHCP Server config file. 
V1.0 211025
Using config backup input file    </path/to/folder/containing/dhcpd_backup/dhcpd-config-pfSense.lan-20211025171029.xml>
Using config backup output file   </path/to/folder/containing/dhcpd_backup/dhcpd-config-pfSense.lan-20211025171029.xml-csv6>
Using CSV master list input file  </path/to/DHCP_master_list.csv>
Warning:  CSV file column name <Active> not found in the <staticmap> template.  This column will be ignored.
Warning:  CSV file column name <WiFi> not found in the <staticmap> template.  This column will be ignored.
Warning:  CSV file column name <Notes> not found in the <staticmap> template.  This column will be ignored.
Processed 27 DHCP static assignments.

```

` `  
## Setup and Usage notes
- Supported on Python3.6+ only.  Developed on Centos 7.9 with Python 3.6.8.  Also tested on Windows 10.
- Tested on pfSense+ version 21.05.1-RELEASE.
- Place the files in a directory on your computer.
- Install lxml module dependency (`pip3 install lxml`), if needed.
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
- dhcpcsv2pfsense builds \<staticmap> blocks in the generated .xml file, one for each line in the .csv file.
- This script may also be run on a full backup file from pfSense.
- You MUST create a _template_ DHCP static mapping entry in any one of your DHCP Server interfaces (does not matter which).  Create this entry with MAC address  `12:34:56:78:90:ab` and with any IP address you like within the interface range.  The script will replace this static mapping entry with the Template defined in your .csv file.  The .xml \<staticmap> block for the Template entry is used as the template for creating all other _real_ \<staticmap> blocks, as defined in your .csv file.
- Field/column names in the .csv that match fields in the \<staticmap> template will overwrite the default values provided from the template.
- You may add columns to the .csv to set values in other fields in the \<staticmap> structure, as needed.
- The order of columns in the .csv file is not important.
- Columns in the .csv that have no match in the \<staticmap> are ignored, with verbose mode warnings.  This feature allows up to have arbitrary columns in the .csv file for supporting notes and comments.  If the field names should change in the pfSense backup and are inconsistent with your .csv file, the script will warn you (with the `--verbose` switch).
- Fields in the \<staticmap> with no matching column in the .csv are kept as in the template - usually blank / empty, with one known exception:  "ddnsdomainkeyalgorithm = hmac-md5".
- Blank lines are permitted in the .csv file, with no output to the generated config file.  _A .csv line is taken as blank if the first column value is blank._  The provided DHCP_master_list.csv file has the first column named `Active` which effectively allows rows to be commented out.
- Double `--verbose` turns on debug level of logging.
- dhcpcsv2pfsense is coded assuming that all of your VLAN subnets have a common IP address prefix, by default `192.168`.  Adjust this regular expression in the code as needed.  The script does not currently support VLANs with a mix of IP prefixes (such as a mix of 10.xxx.xxx.xxx and 192.168.xxx.xx).
         
      IPADDR_FORMAT = r'192.168.([\d]+).\d+'      # Adjust for your local IP subnet strategy


` `  
## Known issues:
- No error checks are performed on the correctness of your .csv file, such as valid IP or MAC addresses, or illegal characters in hostname, for example.  Missing or invalid cell values are taken verbatim, and have unknown impact when loading back into pfSense.  Be careful.
- A backup file .xml from pfSense may have some \<staticmap> fields (i.e. the \<descr> field) using `<![CDATA[]]>` wrappers around the field data.  This script does not generate CDATA wrappers.  The generated .xml file generally loads fine without CDATA wrappers.  Be careful to **not** use xml markup in freeform text lines.  Please open an issue if this breaks.
- As noted above, the script does not currently support VLANs with a mix of IP prefixes (such as a mix of 10.xxx.xxx.xxx and 192.168.xxx.xx).

` `  
## Version history
- V1.0 211025  Reworked to use real .xml handling (lxml dependency).  Supports multiple dhcpd subnets.
- V0.1 211009  Bug fix for row ignore based on first column blank.  Properly carry over the real postamble and block indentation.
- V0.0 211007  New