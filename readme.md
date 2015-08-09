# orientdb-game

[OrientDB](http://www.orientdb.org) is the first Multi-Model Open Source NoSQL DBMS that combines the power of graphs and the flexibility of documents into one scalable, high-performance operational database.

This repository is a dockerfile for creating an orientdb image with :
- explicit orientdb version (orientdb-2.0) for image cache stability
- init by supervisord
- config, databases and backup folders expected to be mounted as volumes

## Build

    bin/build

## Run

    bin/run

## Web Interface

  localhost:2480 (on Mac use `boot2docker ip` instead of localhost)

## Console

    bin/console

