+++
title = "Elasticsearch terminology: Index & Shard & Segment"
date = "2020-11-04"
author = "x1ah"
cover = ""
tags = ["Elasticsearch"]
keywords = ["Elasticsearch"]
description = "Explain of Elasticsearch's terminology: Index / Shard / Segment"
showFullContent = false
+++

References from https://stackoverflow.com/a/15429578

To explain:

## Index

An "index" in Elasticsearch is a bit like a database in a relational DB. It's where you store/index your data. But actually, that's just what your application sees. Internally, an index is a logical namespace that points to one or more shards.

Also, "to index" means to "put" your data into Elasticsearch. Your data is both stored (for retrieval) and "indexed" for search.

## Inverted Index
An "inverted index" is the data structure that Lucene uses to make data searchable. It processes the data, pulls out unique terms or tokens, then records which documents contain those tokens. See http://en.wikipedia.org/wiki/Inverted_index for more.

## Shard
A "shard" is an instance of Lucene. It is a fully functional search engine in its own right. An "index" could consist of a single shard, but generally consists of several shards, to allow the index to grow and to be split over several machines.

A "primary shard" is the main home for a document. A "replica shard" is a copy of the primary shard that provides (1) failover in case the primary dies and (2) increased read throughput

## Segment
Each shard contains multiple "segments", where a segment is an inverted index. A search in a shard will search each segment in turn, then combine their results into the final results for that shard.

While you are indexing documents, Elasticsearch collects them in memory (and in the transaction log, for safety) then every second or so, writes a new small segment to disk, and "refreshes" the search.

This makes the data in the new segment visible to search (ie they are "searchable"), but the segment has not been fsync'ed to disk, so is still at risk of data loss.

Every so often, Elasticsearch will "flush", which means fsync'ing the segments, (they are now "committed") and clearing out the transaction log, which is no longer needed because we know that the new data has been written to disk.

The more segments there are, the longer each search takes. So Elasticsearch will merge a number of segments of a similar size ("tier") into a single bigger segment, through a background merge process. Once the new bigger segment is written, the old segments are dropped. This process is repeated on the bigger segments when there are too many of the same size.

Segments are immutable. When a document is updated, it actually just marks the old document as deleted, and indexes a new document. The merge process also expunges these old deleted documents.


References from https://stackoverflow.com/a/15429578
