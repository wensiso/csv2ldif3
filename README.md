csv2ldif3.pl README
-------------------

This is a brief description of csv2ldif3.pl, a tool written in perl
that you can use to convert CSV files to LDIF files that you can
load into a directory server (like OpenLDAP).

This software is a fork of csv2ldif2.pl with little changes to allow ldif file importation by phpLDAPadmin.
This script was tested succesfully in phpLDAPamin.

Here is the list of differences between csv2ldif3.pl and csv2ldif2.pl:
- In csv2ldif3.pl, we commented the first line of all entries (to avoid the phpLDAPadmin error message: "Error No DN or CONTAINER?").
- In csv2ldif3.pl, we removed quotes for all attribute keys (to avoid the error phpLDAPadmin message: "Invalid syntax (...) An invalid attribute value was specified.").

Also, we added a sample.csv file. In case of necessity, a copy of the original csv2ldif2.pl was included in this repository. 
You can download the file directly from https://sourceforge.net/projects/csv2ldif2/

TODOs:
- We plan to add an option to make csv2ldif3.pl act like the original csv2ldif2.pl script when necessary.

I. BASIC USAGE:

`csv2ldif3.pl -b "ou=example,dc=com" < data.csv > data.ldif`

II. Prerequisites and installing
Installation is not neccessary. Just make the file executable
if it is not already, or run the script through `perl`.
However, before you can run this program, you need:
  * PERL installed (perl.org, on linux probably already installed)
  * PERL modules 'Encode', 'MIME::Base64' and 'Getopt::Std'; all should be core
    modules and already be present in your installation.

III. General
The tool supports many command line options that allow you to fine-tune
the generated LDIF file.
All command line options are shown if you call csv2ldif3.pl without
parameters.
Additional help and some hints is available by calling `csv2ldif3.pl -h`.

Simply spoken, the tool reads CSV data and parses it to LDIF data.
The first line is threaten as header line describing the attributes.
Each records data is mapped to the corresponding attribute.
If you want to write more than one csv-fields data into one
single attribute (multivalue) then you simply need to use the same attribute
name for the affected columns:
--- CSV DATA ---
1:   cn,attr1,attr_mv,attr_mv
2:   foo,test,mv1,mv2

Calling `csv2ldif3.pl -b 'o=example,dc=com'` will parse this into:
--- LDIF DATA ---
1:   dn: cn=foo,o=example,dc=com
2:   attr1: test
3:   attr_mv: mv1
4:   attr_mv: mv2

One of csv2ldif3.pl's features is consolidation of the csv data.
If you have several records for the same entry this is considered.
Only one LDIF entry will be generated for the same dn regardless how often
it will appear in the csv data. As shown below, we have two records
for "foo" (line 2 and 4):
--- CSV DATA ---
1:   cn,attr1,attr_mv,attr_mv
2:   foo,test,mv1,mv2
3:   bar,test2,some_mv1,some_other_mv
4:   foo,test,mv3,mv4

This will generate:
--- LDIF DATA ---
1:   dn: cn=foo,o=example,dc=com
2:   attr1: test
3:   attr_mv: mv1
4:   attr_mv: mv2
5:   attr_mv: mv3
6:   attr_mv: mv4
7:
8:   dn: cn=bar,o=example,dc=com
9:   attr1: test2
10:  attr_mv: some_mv1
11:  attr_mv: some_other_mv


IV. Running and examples
The tool is written to read from STDIN and write to STDOUT (errors go
to STDERR) so you can pipe and redirect if you wish. This gives you
the flexibility to invoke the LDIF generation inside a chain
of different tasks like:

`cat data.csv | head -n 100 | grep -v "foo" | csv2ldif3.pl -b "ou=example,dc=com"`
(convert first 100 lines of data.csv, but ignore records containing "foo".
 Result is printed to standard output for human validation first)

Or another example:
`cat data.csv | head -n 100 | grep -v "foo"
              | csv2ldif3.pl -b "ou=example,dc=com" > data.ldif`
(same as above, but now write LDIF data to file "data.ldif")

Of course you can use it standalone too:
`csv2ldif3.pl -b "ou=example,dc=com" < data.csv > data.ldif`

Have fun!
