SHOPTRACK DATASET

500 dummy online store orders. These are fictional records for practice, not real purchases.
Both files contain the SAME data in different database formats.

FILES IN THIS FOLDER
1. orders_500_couchdb.json - 500 orders prepared for CouchDB bulk import.
2. orders_500_mysql.sql - the same orders prepared for MySQL 8.0 or later.
3. README.txt - this guide.

DATA OVERVIEW
500 orders, 80 fictional customer IDs, 8 products, 1,520 product entries and 5 cities.
Order statuses: processing, shipped, delivered and cancelled.
Each order includes customer ID, items, quantities, prices, totals, address and dates.
Prices are in USD. Tax and shipping charges use fictional example rules.
The structure and product examples are based on JSONSift sample orders:
https://jsonsift.com/datasets/mock-orders
All 500 project orders are generated dummy data.

COUCHDB
1. Start CouchDB and sign in to its web interface.
2. Create a NEW non-partitioned database named shoptrack_500.
3. Open Terminal in the folder containing these files and run:

curl -u admin -H "Content-Type: application/json" -X POST http://127.0.0.1:5984/shoptrack_500/_bulk_docs --data-binary @orders_500_couchdb.json

Enter your local CouchDB password when prompted. Replace admin if your username is different.
Check the returned results for individual errors and confirm that 500 documents were imported.
Import only once into an empty database. Repeating the import can cause document conflicts.
This file contains multiple documents; do not paste the entire file into Create Document as one order.

MYSQL
1. Start your local MySQL server and open MySQL Workbench.
2. Open orders_500_mysql.sql.
3. Confirm that a database named shoptrack_500 does not already exist.
4. Run the whole script once and check Action Output for errors.
5. Refresh Schemas. The script creates shoptrack_500 with two tables:
   orders: 500 rows.
   order_items: 1,520 rows, linked to orders by order ID.
If the database already exists, stop and inspect it before continuing. Do not rerun the inserts into existing tables.

WHY TWO VERSIONS
CouchDB stores each order and its items together in a JSON document.
MySQL stores orders and their items in related tables.
We can use the same data to compare storing, adding, finding, updating and deleting information.
Changes in one database do not automatically update the other database or another laptop.
Everyone should start with the same files.

The data structure, arithmetic and matching representations have been checked.
Importing the files into running CouchDB and MySQL servers still needs to be tested by the team.
