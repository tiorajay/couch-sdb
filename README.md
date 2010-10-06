# Couch-SDB

This is an SDB back-end to the venerable [BIND](http://www.isc.org/software/bind) name server. It implements an interface to the [CouchDB](http://couchdb.apache.org/) database, which is a distributed, schema-free document database.

## Why?

Because it's cool. Because CouchDB replicates over port 80, whereas zone transfers (AXFR) use port 53. Because CouchDB is proxyable. Because it is fast enough for moderate use. And mainly, because I felt like doing it ever since I created a proof of concept [using a pipe back-end to PowerDNS](http://blog.fupps.com/2010/05/05/powerdns-and-a-couchdb-backend/).

## Does it... ?

Yes, it does. A bit. Speed isn't bad at all. Due to limitations of the Bind SDB back-end, `couch-sdb` cannot possibly be faster than <insert-your-favorite-driver-here>. 

The CouchDB portion doesn't use a view in any way. It's either the document for the zone is there, or it ain't. My first version queried a view to find the requested domain name, but I was getting an unecessary number of hits on the database per query, so I scrapped that.


## Requirements

1. [cdbc](http://github.com/jpmens/cdbc)
2. [Jansson](http://www.digip.org/jansson/)

## Building

1. Read my chapter on Bind's SDB! :)
2. Patch `bin/named/Makefile.in` with

		DBDRIVER_OBJS = couch-sdb.o 
		DBDRIVER_SRCS = couch-sdb.c 
		DBDRIVER_INCLUDES = 
		DBDRIVER_LIBS = -lcdbc -lcurl -ljansson

3. Fix `bin/named/main.c` as per my book.
4. `./configure && make`

## Testing

1. Create a minimal `named.conf` containing something like this:

		controls {
		};

		options {
		    directory "/tmp";
		    listen-on port 9953  { 127.0.0.1; };
		    listen-on-v6 {none;};
		    allow-query {any;};
		};

		zone "example.org" {
		    type master;
		    database "couch http://couch-server.com:5984 dns";
		                ^           ^                    ^^^
		                |           |         	          |
		                |           | 	                  + dbname
		               	|           +---------------------- URL
		               	+---------------------------------- keyword
		};

2. Launch `named` in foreground

		./named -4 -n 1 -d 1 -g -c jp.conf

3. Query it

		dig -p 9953 @127.0.0.1 www.example.org
