# System Design Engineering Journal and Notes

## Tuesday, September 10, 2019

### Challenges Faced
* Becoming acquainted with legacy codebase

### Actions Taken
I read through the existing codebase to become familiar with what data was passed from client to server to database and back.

### Results/Takeaways
It was very easy and straightforward to work through the code. I will attempt to keep this code clean and readable.


## Wednesday, September 11, 2019

### Challenges Faced
* Designing schema for PostgreSQL
* Generating 10M records and efficiently loading into database (ideally in under 10 minutes, 1 hour tops)

### Actions Taken
The item description module displays the following information: product name (string), price (number), number of reviews (number), number of questions asked (number) and related images (array). In addition to this is an ID that is not publically shown but available to use as an identifier to access items. I will also use product names to query the database. Below are the schemas I have designed.

| Products                         |
| -------------------------------- |
| id (int PRIMARY KEY FOREIGN KEY) |
| name (varchar(200))              |
| price (int)                      |
| num_reviews (int)                |
| num_questions (int)              |

| Colors                | red | invisible | blue | green | yellow | orange | purple |
| --------------------- | --- | --------- | ---- | ----- | ------ | ------ | ------ |
| id (int FOREIGN KEY)  |     |           |      |       |        |        |        |

Each column in the colors table will store a boolean denoting availability. I created a database and 2 tables in the format above.

I installed and used Faker.js to generate 1 million records, using the npm package, csv-writer to create a CSV file from an array.

Time (in milliseconds) taken to generate 1M records
![Time taken to generate 1M records](https://imgur.com/nHImTwA 'Time taken to generate 1M records')
Note: there was no existing CSV file for the first time. The others are rewriting the existing file.

In my first attempt to populate the database, I consulted PostgreSQL documentation which recommended turning off autocommit (off by default), using `COPY` to load, dropping indexes and keys and manipulating the `maintenance_work_mem` and `max_wal_size` properties. Per documentation, `COPY` incurs a significantly less overhead for large data loads in comparison to `INSERT`.

To do this, I created a .sql file to create a database and the first table. I removed the primary key index and foreign key constraint to optimize bulk data loading and plan to recreate these after all data is loaded. When data is added to a table with an existing foreign key constraint, each new row is entered on the server's list of pending trigger events which can cause the trigger event queue to overflow available memory for large sets of data. Note, however, that there is a caveat to removing constraints which is the loss of error checking when these constraints are missing. I also ran `ANALYZE` once seeding was complete to ensure the planner has up-to-date statistics regarding the table.

At this time, I have not tried the other suggestions of increasing `max_wal_size` when loading data (which reduces the number of checkpoints required) and increasing `maintenance_work_mem` when manually recreating indices and foreign keys afterwards.

### Results/Takeaways
When seeding data, I ran into a 'missing data for column' error that stumped me for a few hours. I realized my SQL `COPY` query had an incorrect file path. Double check code to ensure there are no typos and obvious errors.


## Thursday, September 12, 2019

### Challenges Faced
* Generating a CSV file with 10M records with which to seed database

### Actions Taken
Because I need a metric to determine the efficiency of seeding, I decided to load PostgreSQL through a POST request with Express. I plan to use the npm module, timer-node, to measure the speed of all tasks.

### Results/Takeaways
I had difficulties connecting to the database through the node-postgres module. Initially, I specified the incorrect username in the config and was not error handling which made it extremely frustrating to pinpoint the location of my error. I also realized that the queries required either a callback or promise which I was missing. After chaining a series of functions starting with `.connect`, I was able to make a successful GET request. However, I am getting errors when attempting to write to my table.


## Friday, September 13, 2019

### Challenges Faced
* Seeding database with primary records
* Generating second CSV file and table of colors and sellingPoints with 10M records each

### Actions Taken
I seeded PostgreSQL with 10M primary records. Then, I decided to change the schema for the colors table after realizing that it is storing a lot of unnecessary data. See below.

Colors
| id | color (varchar(20))  | products_id (FOREIGN KEY) |
| -- | -------------------- | ------------------------- |
|    |                      |                           |

I also seeded with the secondary records and added a third table to store descriptive bullets for each item which I had previously forgotten.

### Results/Takeaways
It took 27s to load the database with 10M items through a POST request with Express. Perhaps this is faster if queries are made directly to PostgreSQL. It took 262ms to generate a CSV file of 10M secondary items and 20s to seed the database with said items.


## Saturday, September 14, 2019

### Challenges Faced
* Creating a different CSV file to load second database
* Seeding second database with 10M records

### Actions Taken
Because I could not generate 10M records without a JavaScript heap out of memory error, I generated records and seeded using `bulkWrite` in increments of 1M, taking approximately 1 minute for each cycle.

### Results/Takeaways
My seeding approach is far from ideal, so I plan on revisiting this, using either `mongoimport` with a separate CSV file or farm which allows you to create as many processes as number of CPUs available, exceeding the standard single core for the Node.js environment. `mongoimport` is convenient because it is a prewritten import tool that can be utilized to load Mongo; however, it does not give the user the ability to transform documents. It is also single-threaded, so multi-threaded bulk inserts eventually surpass `mongoimport`.


## Monday, September 16, 2019

### Challenges Faced
* Querying PostgreSQL

### Actions Taken
I made queries to get baseline times then indexed columns and requeried PostgreSQL. I also used `EXPLAIN ANALYZE` to view the processed time of queries.


### Results/Takeaways
I queried the products table in PostgreSQL using product IDs 1, 10000000 and 5000000 which took 3.616s, 0.923s and 0.794s, respectively. When querying to all 3 tables simultaneously. it took much longer at 21.688s for product 1 and 14.393s for product 10000000. In my first attempt to lower query times, I added indexes to the product ID column of the products table. This sorts the table by indexed column and performs a binary search to look for the row(s) of interest instead of a sequential scan. Query times were 8.370s for product 1000000 and 8.570s for product 10000000. Far from optimal, I added another index to the product ID column of the colors table which resulted in the following query times for the same products: 5.203s and 5.762s. After adding an index to the last table, queries improved significantly to 0.20s, 0.17s, 0.15s and 0.21s for randomly selected items. `EXPLAIN ANALYZE` showed query planning times of 0.497ms, 0.135ms, 0.127ms and 0.135ms and execution times of 5.707 ms, 3.364ms, 3.405ms, 0.129ms for items 1, 10000, 1000000 and 10000000.

When implementing indexes, it is important to acknowledge the trade off between storage space and query time.
indexing can introduce overhead for data manipulation
when read query performance is a priority, indexing is good approach


## Tuesday, September 17, 2019

### Challenges Faced
* Querying MongoDB

### Actions Taken
I made queries to get baseline times then indexed columns and requeried MongoDB.

### Results/Takeaways
It took 63.359s, 36.641s, 64.770s and 55.983s to query for products 1, 100, 1000 and 1000000 respectively. Similarly to PostgreSQL, MongoDB scans an entire collection to find matching documents when querying without an index. I used indexes on the product ID field to limit the number of documents that are searched in the collection. Queries improved to 0.78s, 0.1s, 0.7s and 0.15s for the same products.


## Wednesday, September 18, 2019

### Challenges Faced
* Researching tools to stress test server

### Actions Taken

### Results/Takeaways


## Thursday, September 19, 2019

### Challenges Faced
* Load testing on local machine

### Actions Taken

### Results/Takeaways