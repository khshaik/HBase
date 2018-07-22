# HBase

Deminstifying Columnar databases:
---------------------------------
Overview of Bigdata, Hadoop Architecture.
Evolution of NOSQL.
Classification of NOSQL databases.
Column-oriented databases.
Google's Big table.
Diff: Row-oriented databases & Column oriented databases.
HBase vs. RDBMS.
HBase.

Introduction:
--------------
Relation database not sufficient to meet data storage and processing needs.
Not distributable across multiple nodes for distributed processing.
Object oriented database => Flop.
Deal with ever-increasing data => Scale vertically i.e. Employ high-end servers and increase RAM for processing.
High-costs, Network delays while transferring bulk of data from server to client.

Retrieve all of the American league teams.
Traditional database rules: Reading complete row from top to bottom. In each row, fetching data of only two columns is not efficient.
Columnar database rules: Read the data from top to bottom and only the columns needed.

Research at google for about 13 years => GFS, 2003.
One year later processing paradigm => MapReduce, 2004.

Yahoo => Hadoop, 2006
Yahoo => MapReduce, 2006

Donated to Apache project and is currently distributed under Apache's license.


Big table:
----------
Bigtable is a distributed storage system for managing structured data that is designed to scale to a very large size: petabytes of data across thousands of commodity servers. Many projects at Google store data in Bigtable, including web indexing, Google Earth, and Google Finance. These applications place very different demands on Bigtable, both in terms of data size (from URLs to web pages to satellite imagery) and latency requirements (from backend bulk processing to real-time data serving). Despite these varied demands, Bigtable has successfully provided a flexible, high-performance solution for all of these Google products. In this paper we describe the simple data model provided by Bigtable, which gives clients dynamic control over data layout and format, and we describe the design and implementation of Bigtable.

Classification of NOSQL databases:
----------------------------------
Key-Value: Data stored in the form of Key & Value pairs.
Big table: Data is stored in columar fashion where columns are grouped together in column families.
Document: Data stored in the form of document. JSON.
Graph: Suited for graph representation of data. Neo4j. Facebook, LinkedIn. Find connection among nodes and edges.

Apache Accumulo — built on top of Hadoop, ZooKeeper, and Thrift. Has cell-level access labels and a server-side programming mechanism. Written in Java.
Apache Cassandra — brings together Dynamo's fully distributed design and BigTable's data model. Written in Java.
Apache HBase — Provides BigTable-like support on the Hadoop Core.[15] Has cell-level access labels and a server-side programming mechanism too. Written in Java.
Hypertable — Hypertable is designed to manage the storage and processing of information on a large cluster of commodity servers.[16] Written in C++.
"KDI", Bluefish, GitHub — Kosmix attempt to make a BigTable clone. Written in C++.
LevelDB — Google's embedded key/value store that uses similar design concepts as the BigTable Tablet.

HBase: Not for OLTP
------
Distributed, Scalable, Column-oriented datastore on top of Hadoop/HDFS. Millions of columns * Billions of Rows 

History: Modelled after Google's Big table paper. Written in Java.
		 Developed as part of Apache Hadoop project.
		 Fault tolerant way of storing large quantities of data.
         Real-time column-oriented database.

Purpose: Provides access/updates to very very large data stored in HDFS possible in couple of seconds.
		 Huge data, Fast random access, Structured data, Schema is flexible, Need of compression, Need of distribution (Shrading/ Partitioning), Sparse, Consistent, Distributed, Sorted map, Multi-dimensional.
		 Data can be stored on over large number of commodity servers.
		 One does not define number and types of columns upfront.

Storage: Stored on top of HDFS in Hadoop.
         Everything is stored in the form of byte-array.
		 
Interface access: Shell, REST, Thrift API, JVM Client, MapReduce, Avro.

Loading data into HBase: Import/Export from RDBMS using Sqoop ?..

Row store vs. Columnar store:
-------------------------------
In row store, RDMS assigns unique key to each row and stored entire row on a single disk. 
Designed for efficient and fast single read/write operations but falls short on aggregating, summarizing entire data set.
Pre-build composite indexes of different sets of columns in order to have all sets of columns.

In columnar store, column values are stored together. Mainly designed for OLAP queries.
Key-Value pair model efficient for reading data on entire data set on a small set of columns.
Column data is stored in a single block and hence aggregation on columns is very much faster.
High degree of compression due to few distinct values or sparse data.
Not efficient for reading entire row data especially in case of large number of columns.


Rowkey = Primary key, Column family = One level hierarchy groups multiple columns, 
       Column = Column in RDBMS [Identified by Column family & Column name], Timestamp = Timevalue when data was inserted or modified automatically maintained by HBase,
	   Columns in a column group can overlap.
	   
Why HBase?.
-----------
			HBase                   Hadoop
-----------------------------------------------------------
Real-time access to data        Batch-oriented
Low latency (Performance)       High latency (User submits jobs and is notified after job completion)
Very efficient in reading       To read entire data-set
single to large range of rows
from billions of rows
Random real-time model          Write once read-only model

Fast, random, real-time, read/write access to peta bytes of data.

HBase Architecture and Working:
--------------------------------
Stores its data files on top of Hadoop. 
Leverages all benefits of Hadoop i.e. Distributed, Fault-tolerant, Horizontally scalable, Runs on commodity hardware.
Designed to handle vast amount of data to the natures of peta-bytes.
Uses zoo keeper as a co-ordination service.
Follows Master/Slave pattern where Master monitors and assigns Region servers or Slave nodes.
Master is not on the data path and is not a bottleneck to client queries, actual work is done by Slave nodes/ Region servers
which are responsible for managing one or more regions.
Provides auto-shrading or partitioning of data into regions as the data grows.
Each table is partitioned into equal number of partitions called Region. 
Each region is assigned to a Region server across the cluster which can hold regions of different tables.
Each region server holds roughly equal number of regions.

Region Server: Each Region server contains single Block cache and single HLog (WAL-Write Ahead Log) and multiple regions.
Block cache:   Block cache supports read operations and holds any data which is needed by queries for a short period of time.
MemStore:      MemStore holds in-memory modifications to the table until it is flushed to HFile. Stores data in the form of Key-Value pair.
               Each Region server writes updates such as put/delete to WAL and then to MemStore before MemStore gets fluch to HFiles. This
               is for durability i.e. if Region server were to go down before writing to HFile, the data is still available in WAL log.
Regions:       Regions are the basic elements of availability and distribution for tables, these are comprised of a store called Column 
               family.
Meta:		   Mapping of Regions to Region servers is stored in a system table called Meta.
Read/Write:    When trying to read/write data from HBase, clients read required amount of information from the Meta table and
               communicate directly with appropriate region server.
HStore:        Each Region server has multiple HStores for column family of table. HStore holds zero to many HFiles and a MemStore.

HFile:         HFile is ultimately where the data lives.

Data Model:
------------
Tables:  Declared at schema definition time, stored stored by row keys and every thing in the table is stored as bytearrays and there are
         no types.
Row keys: Are byte arrays which are sorted lexicographically. Indexing is by row-key only. Atomicity is guaranteed is at row-level and not
          at column level, thus transactions are not supported.
Column families: Group of columns. Declared at the time of table creations. Each table should have atleast one column family.
Columns: Identified by column family and qualifier.
Timestamp: System time on the write to cell.
Each Cell is uniquely identified by rowkey, column, timestamp/version.

Example:
--------
Dataset is a daily view of stocks performance of 4 publicly traded companies.

HBase vs. RDBMS:
-----------------
	     HBase                      		RDBMS
-------------------------------------------------------------------------
1. Column-oriented          			Row-oriented
2. Schema-less              			Schema on write
3. Tall and wide (Billions of rows      Short and thin (Millions of rows
   across millions of columns)          across hundreds of columns)
4. Scales horizontally (Add more        Scales vertically (Add more
   nodes and machines)                  CPUs and RAM)
5. Used for real-time and read/write    Used for OLTP based transactions
   access to analytical queries
6. No support for SQL or SQL-like       Supports ANSI-92 SQL specification
   language
7. De-normalized						Highly Normalized

New SQL:
--------
NewSQL is a class of modern relational database management systems that seek to provide the same scalable performance of NoSQL systems for online transaction processing (OLTP) read-write workloads while still maintaining the ACID guarantees of a traditional database system.

Questions:
----------
Identification of each value.
Column part of more than one column family.
Changing table structure later.

Videos:
-------
CAP: http://www.youtube.com/watch?v=Jw1iFr4v58M
Edureka: https://www.youtube.com/watch?v=fg0RMflDFPI
Gregory: https://www.youtube.com/watch?v=jplBWzFKA0g
Martin Flower: https://www.youtube.com/watch?v=qI_g07C_Q5I
http://martinfowler.com/bliki/PolyglotPersistence.html
http://www.odbms.org/wp-content/uploads/2014/04/Chapter-13-Polyglot-Persistence.pdf
http://martinfowler.com/nosql.html

HBase Schema Design: https://www.youtube.com/watch?v=_HLoH_PgrLk
Column oriented databases: https://www.youtube.com/watch?v=mRvkikVuojU
NOSQL Databases: http://nosql-database.org/
Durga Soft: https://www.youtube.com/watch?v=VEmy3I5eq74
Havard: https://www.youtube.com/watch?v=dty9uu5gW6w
Inctricity: https://www.youtube.com/watch?v=8KGVFB3kVHQ
InfiniDB: https://www.youtube.com/watch?v=wYl_YxqTof4
Row vs. Column vs. NOSQL: https://www.youtube.com/watch?v=ja7qkg8-GYw
CAP Theorom: https://www.youtube.com/watch?v=ytPoGSNV8z8

Wiki's:
--------
Column-oriented DBMS: http://en.wikipedia.org/wiki/Column-oriented_DBMS
Databases in Era of Cloud Computing & Big Data: http://www.opensourceforu.com/2011/05/databases-in-era-of-cloud-computing-and-big-data/
NewSQL — The New Way to Handle Big Data: http://www.opensourceforu.com/2012/01/newsql-handle-big-data/
Importance of In-memory databases: http://www.opensourceforu.com/2012/01/importance-of-in-memory-databases/
Impact of Bigdata on Enterprise integration: http://www.opensourceforu.com/2012/04/impact-of-big-data-on-enterprise-integration/
16 NOSQL, NewSQL databases to watch: http://www.informationweek.com/big-data/big-data-analytics/16-nosql-newsql-databases-to-watch/d/d-id/1269559
Datavail: http://www.datavail.com/category-blog/what-is-newsql/
SQL comes back to NOSQL: http://www.infoq.com/news/2013/11/sql-newsql-nosql
Column vs. Row-oriented databases: https://www.youtube.com/watch?v=doQfVHGfFJA
NOSQL, NewSQL, Traditional DBMS: http://www.dbms2.com/2014/03/28/nosql-vs-newsql-vs-traditional-rdbms/
New SQL: http://en.wikipedia.org/wiki/NewSQL, http://www.slideshare.net/hypermin/newsqldatabaseoverview

Setup:
-------
cd /usr/local/hadoop
bin/start-dfs.sh
bin/map-red.sh


cd hbase-0.94.0/bin
> start-hbase.sh
> hbase shell

Commands:
----------
1. create 'tablename', 'columnfamily'.
2. describe 'tablename'.
3. scan 'tablename'.
4. Select some columns by rowkey: get 'tablename, 'rowid', ['columnfamily:col1', 'columnfamily:col2', 'columnfamily:col3']
	get 'stocks', 'YAHOO_2', ['perf:Date', 'perf:Close', 'perf:Volume', 'perf:Open', 'perf:High', 'perf:Low']
	get 'stocks', 'GOOGLE_1', ['perf:Symbol', 'perf:High', 'perf:Low']
5. Select all columns of a table by rowkey: get 'tablename', 'rowid', 'columnfamilyname'
   get 'stocks', 'YAHOO_2', 'perf'
6. Put data into table based on rowkey: 
      put 'tablename', 'rowid', 'columnfamily:col1', 'value1'
	  put 'tablename', 'rowid', 'columnfamily:col2', 'value2'
	  put 'stocks', 'YAHOO_2', 'perf:High', '560.02.3'

UseCase1:
---------
1. Create a simple table using command create 'test', 'data'
2. Verify the table exists using the command list
3. Insert data into the table using e.g.
put 'test', 'row1', 'data:1', 'value1'
put 'test', 'row2', 'data:2', 'value2'
put 'test', 'row3', 'data:3', 'value3'
4. List all rows in the table using the command scan 'test'.  Notice how 3 new columns where added without changing the schema!
5. We get rid of table by issuing disable 'test' followed by drop 'test' and verified by list which should give an empty listing.

UseCase2:
---------
Rowkey 	  Symbol 	  Date 		Close 	Volume 	    Open    High 	Low
---------------------------------------------------------------------------
GOOGLE_1  GOOGLE 	1/21/2015 	89.54 	67890123 	61.5 	67.8 	64.5
GOOGLE_2  GOOGLE 	1/22/2015 	88.76 	66890771 	60.4 	66.3 	62.7
YAHOO_1   YAHOO		1/21/2015 	551.01 	4345677 	547.89  556.88  550.02
YAHOO_2   YAHOO 	1/22/2015 	505.06 	4567890 	556.67  560.02  502.99
CROCIN_1  CROCIN	1/21/2015 	12.56 	7654321 	12.45   12.89   12.32
TATA_1 	  TATA 		1/21/2015 	178.22 	113456 		176.44  179.34  172.33
TATA_2 	  TATA 		1/22/2015 	190.56 	116789 		192.46  195.56  191.11
BMW_1 	  BMW 		1/21/2015 	250.01 	256789 		249.99  253.38  248.87
BMW_2 	  BMW 		1/22/2015 	252.03 	257786 		248.77  259.19  251.14
MERK_1 	  MERK 		1/21/2015 	124.67 	3678912 	121.09  124.56  119.87
MERK_2 	  MERK 		1/22/2015 	99.99 	3287901 	122.22  124.56  120.03

create 'stocks', 'perf'
describe 'stocks'

put 'stocks', 'GOOGLE_1', 'perf:Symbol', 'GOOGLE'
put 'stocks', 'GOOGLE_1', 'perf:Date', '1/21/2015'
put 'stocks', 'GOOGLE_1', 'perf:Close', '89.54'
put 'stocks', 'GOOGLE_1', 'perf:Volume', '67890123'
put 'stocks', 'GOOGLE_1', 'perf:Open', '61.5'
put 'stocks', 'GOOGLE_1', 'perf:High', '67.8'
put 'stocks', 'GOOGLE_1', 'perf:Low', '64.5'

put 'stocks', 'GOOGLE_2', 'perf:Symbol', 'GOOGLE'
put 'stocks', 'GOOGLE_2', 'perf:Date', '1/22/2015'
put 'stocks', 'GOOGLE_2', 'perf:Close', '88.76'
put 'stocks', 'GOOGLE_2', 'perf:Volume', '66890771'
put 'stocks', 'GOOGLE_2', 'perf:Open', '60.4'
put 'stocks', 'GOOGLE_2', 'perf:High', '66.3'
put 'stocks', 'GOOGLE_2', 'perf:Low', '62.7'

put 'stocks', 'YAHOO_1', 'perf:Symbol', 'YAHOO'
put 'stocks', 'YAHOO_1', 'perf:Date', '1/21/2015'
put 'stocks', 'YAHOO_1', 'perf:Close', '551.01'
put 'stocks', 'YAHOO_1', 'perf:Volume', '4345677'
put 'stocks', 'YAHOO_1', 'perf:Open', '547.89'
put 'stocks', 'YAHOO_1', 'perf:High', '556.88'
put 'stocks', 'YAHOO_1', 'perf:Low', '550.05'

put 'stocks', 'YAHOO_2', 'perf:Symbol', 'YAHOO'
put 'stocks', 'YAHOO_2', 'perf:Date', '1/22/2015'
put 'stocks', 'YAHOO_2', 'perf:Close', '505.06'
put 'stocks', 'YAHOO_2', 'perf:Volume', '4567890'
put 'stocks', 'YAHOO_2', 'perf:Open', '556.67'
put 'stocks', 'YAHOO_2', 'perf:High', '560.02.3'
put 'stocks', 'YAHOO_2', 'perf:Low', '502.99'

Questions:
-----------
What is the oldest volume for a given symbol?
scan 'stocks', {COLUMNS=>['perf:Symbol','perf:Date','perf:Volume'],LIMIT=>1,STARTROW=>'GOOGLE_1'}

What is the stock performance in the past two days?
scan 'stocks', {LIMIT=>2,STARTROW=>'GOOGLE_1'}

What is the latest high/low by Stock?
scan 'stocks', {COLUMNS=>['perf:Symbol','perf:Date','perf:High','perf:Low'],LIMIT=>1,STARTROW=>'GOOGLE_2'}

What is the performance of stock on '1/22/2015'.
scan 'stocks', {COLUMNS=>['perf'],FILTER=>"(SingleColumnValueFilter('perf','Date',=,'regexstring:1/22/2015',false,false))"}
select * from stocks where date like '1/22/2015'


Commands:
-----------
$ net stop sshd
$ cygrunsrv -R sshd
$ net user sshd /DELETE  # See note below
$ rm -R /etc/ssh*
$ mkpasswd -cl > /etc/passwd
$ mkgroup --local > /etc/group

$ editrights -l -u kashai

$ editrights.exe -a SeAssignPrimaryTokenPrivilege -u kashai
$ editrights.exe -a SeCreateTokenPrivilege -u kashai
$ editrights.exe -a SeTcbPrivilege -u kashai
$ editrights.exe -a SeServiceLogonRight -u kashai

$ ssh-host-config

$ net start sshd