<h3>Google Cloud BigTable</h3>

<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Powers many core Google services including Search, Analytics, Maps, and Gmail
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Managed NoSQL (However it's NOT no-ops. You must configure nodes)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Designed for Low-Latency, High-Throughput
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Key-Value structure based on Row Key (similar to Apache HBase)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Petabyte-Scale (Best with 1TB+ of data)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Auto-Scaling Storage
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Deploys in a selected REGION (ie. us-east1) - Cross region replication in Beta
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Tables can be sparse (sparse cell takes up no space) 
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Can you CLI to interact with BigTable (cbt or hbase shell)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;BigTable learns over time (re-distributes reads, re-distributes storage)
<br>
<br><b>Concepts</b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Each table has only one index, the ROW KEY. There are no secondary indices.
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Column family - grouped columns that are related to one another
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;column qualifier - unique name within the column family
<br>
<br><b>Modes</b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Production (3+ nodes, HA, CANNOT downgrade)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Development (Can upgrade anytime to Production)
<br>
<br><b>Storage</b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Selection of Storage is permanent
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Changing disk type (ie. HDD to SSD) requires new instance
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Auto-Scaling Storage
<br>
<br><b>Architecture</b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;No data is stored on the node. Pointers (metadata) is stored on the nodes
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Each row is indexed by a single row key
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Columns related to one another are typically grouped together into a column family
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Each column is identified by a combination of the column family and a column qualifier
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Cloud Bigtable tables are sparse
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;If a cell does not contain any data, it does not take up any space.
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;A Table is sharded into blocks of contiguous rows, called tablets, to help balance workload of queries
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Tablets are stored on Colossus, Google's file system, in SSTable format
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Data is never stored in Cloud Bigtable nodes themselves
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Each node has pointers to a set of tablets that are stored on Colossus.
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;How is data organized?
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;One big table (called an instance)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;This table is sharded across tables
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Nodes contain metadata that points to data (which is stored on Collosus)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Table can be thousands of columns
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Table can contain billions of rows
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Indexing and Queries
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Only the row key is indexed
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Row Keys are stored in ascending order
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;All operations are atomic at the row level
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Schema design is necessary for efficient queries
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Field promotion - Good key design strategy, uses field(s) from column data in row key
<br>
<br><b>Schema Design</b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Per Table - row key is the only indexed item
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Keep all entity info in a single row
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Related entities should be in adjacent rows
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Tables are sparse (empty columns take up no space)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Good Row Keys:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Reverse domain names (com.google.support)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;String identifies 
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Timestamps (reverse, NOT at front/or only identifier)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Bad Row Keys:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Domain names (support.google.com)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Sequential IDs
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Timestamps alone or at front
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Frequently updated identifiers
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Hashed values (no longer a recommended practice)
<br>
<br><b>Performance</b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Performance (SSD)	10,000 QPS @ 6 ms latency (for reads and writes)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Causes of poor performance
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Poor schema row key design
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Workload isn't appropriate for Cloud Bigtable
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Rows in your Cloud Bigtable table contain large amounts of data
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Rows in your Cloud Bigtable table contain a very large number of cells.
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Cluster doesn't have enough nodes
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Cluster was scaled up or scaled down recently
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Cluster uses HDD disks
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Instance is a development instance
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Issues with the network connection
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Testing performance with Cloud Bigtable
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Use a production instance
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Use at least 300 GB of data
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Stay below the recommended storage utilization per node
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Before you test, run a heavy pre-test for several minutes
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;Run your test for at least 10 minutes
<br>
<br><b><a href="https://cloud.google.com/bigtable/docs/access-control">IAM and Security</a></b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Access control at the project level and the instance level
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Predefined Roles
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;roles/bigtable.admin
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;roles/bigtable.user (read-write access, intented for developers)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;roles/bigtable.reader (read-only access to the data stored within tables)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&ndash;&nbsp;roles/bigtable.viewer (Provides no data access)
<br>
<br><b>Use Cases</b>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Time-series data (tall and narrow table)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Marketing data - purchase histories and customer preferences
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Financial data - transaction histories, stock prices, and currency exchange rates
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;IoT - usage reports from energy meters and home appliances
<br>&nbsp;&nbsp;&nbsp;&nbsp;&bull;&nbsp;Graph data - info about how users are connected to one another
<br>
