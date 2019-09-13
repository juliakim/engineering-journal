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

Colors
|                       | red | invisible | blue | green | yellow | orange | purple |
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