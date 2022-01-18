This package based on tpc-e workload described 
on http://www.tpc.org/tpce/default.asp
Introduction
=============
This is a modified version based on the TPC-E version from Percona https://www.percona.com/blog/2010/02/08/introducing-tpce-like-workload-for-mysql/.
The package is based on the TPC-E workload described on http://www.tpc.org/tpce/default.asp
and uses EGen software provided by TPC. 

In this package SAP HANA Support was added to the TPC-E Percona version.

Installation
============
Before you can build the tool you need to install following dependencies:
- unixodbc driver (>= 2.3.4)
- ODBC driver for HANA (Linux) (Tested with 64 bit version of driver)
- LLVM
- CLANG



Install unixodbc from Source (>= 2.3.4)
------------------------------
You have to install unixodbc from source instead of using package manager installed on your system (e.g. homebrew), since the package 
homebrew is providing is outdated and missing some crucial fixes introduced in 2.3.3.

- Get source from http://www.unixodbc.org/ and untar it into an directory
- Run `./configure` on your system
- Run `make`
- Run `make install` (sudo maybe required here to copy libraries into system path)

Compile TPC-E
--------------
- Adjust the include path in the `makefile` in the directory `prj` to the directory with the unixodbc library / include files (if not located in the usual system directories) on your system
- Execute the makefile by calling `make`


Usage
=====
Generate Test Data 
----------------------------------------
First you have to generate Test Data using `EGenLoader` in the bin directory supplying the path to the `flat_in` and `flat_out` directory. You can adjust the generator by supplying additional arguments to control the size of the generated dataset. 

```
./EGenLoader -i flat_in -o flat_out -c <number_of_customers> -t <active_customers> -f <number_of_customers_for_1TRTPS> -w <days_of_trade>
```

Concrete Example:
```
./EGenLoader -i flat_in -o flat_out -c 2000 -t 2000 -f 200 -w 50
```

After successfull generation of the benchmark data the generated data is located in the `flat_out` directory.

Prepare Database for Loading of the Flat Files
---------------------------------------------
First you have to create the tbales for the TPC-E like benchmark by executing the SQL script `1_create_table.sql` located in the directory `scripts/<your_database>`.

Load Flatfiles into SAP HANA with hdbsql
----------------------------------------

Before you execute can import the generated flat files you have to generate the SQL import script in the directory `scripts/<your_database>` (only SAP HANA). To generate the load script execute the python file `python 2_load_data.py <path_to_flat_files> <schema_name>` supplying the path to the direcotry containing the flat files and the schema name.

We use the hdbsql command line tool supplied with the HANA ODBC driver to load the generated data into the SAP HANA database.
We load the flatfiles into the SAP HANA database by executing following command:
```
<path_to_hdbsql_executable>/hdbsql -n <IP_HANA_INSTANCE>:3<INSTANCE_ID>15 -u <USERNAME> -p <PASSWORD> -I <PATH_TO_IMPORT_SCRIPT>/hana_csv_import.sql -c ';'
```

**Please confirm that the User used for the import has the user rights to IMPORT / INSERT data into the SCHEMA/TABLE!**

Create Indicies / ... in Database
-----------------------------------------
After successfully importing the previously generated flat_files you have to execute the remaining SQL scripts (3,4) in ascending order.


Execute TPC-E like Benchmark
-----------------------------
To execute the Benchmark run the following command:
```
./EGenSimpleTest -c <number_of_customers> -a <active_customers> -f <number_of_customers_for_1TRTPS> -d <days_of_trade> -l <number_of_customer_load_unit> -e <path_to>/flat_in -D <name_of_data_source> -U <username> -P <password> -t <duration> -r <ramp_up> -u <number_of_users>
```

Example
```
./EGenSimpleTest -c 2000 -a 2000 -f 200 -d 50 -l 200 -e ../flat_in -D hana -U <user> -P <password> -t 30 -r 10 -u 2
```
Usage Example for Percona TPC-E like benchmark:
https://www.percona.com/blog/2010/02/09/introducing-percona-patches-for-5-1/

**The values `<number_of_customers>`,`<active_customers>`,`<number_of_customers_for_1TRTPS>`,`<days_of_trade>` have to be the same as used at the ./EGenLoader data generator !***

License
=======
This package based on tpc-e workload described  on http://www.tpc.org/tpce/default.asp
and uses EGen software provided by TPC.

The package is fully compatible with
TPC license
http://www.tpc.org/tpce/egen/TPC-E%20License%20Agreement.pdf
The results you get with this package
can't be named "TPC Benchmark Results"
and are not compatible or comparable with
TPC-E Benchmark results 
