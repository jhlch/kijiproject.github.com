---
layout: post
title : SongCount
categories: [tutorials, music-recommendation, 1.0.0-rc4]
tags: [music]
order : 4
description: Like WordCount, for songs.
---


### The 'Hello World!' of MapReduce
To quote Scalding Developers ['Hadoop is a distributed system for counting words.']
(https://github.com/twitter/scalding) Unfortunately, we here at PANDORA have very few words to
count, but millions of users that play millions of different songs.

This MapReduce job uses the listening history of our users that we have stored in a Kiji table to
calculate the total number of times each song has been played. The result of this computation is
written to a sequence file in HDFS.

### SongPlayCounter.java
The SongPlayCounter is an example of a [Gatherer](link to gatherer docs), which is essentially a mapper
that gets input from a KijiTable.

reads input from a Kiji Table and outputs 
KijiRowData -> <SongID, 1>

* getDataRequest()
A gatherer takes input from a table, so it must declare what data it will need. It does this in the
form of a [KijiDataRequest](link data request + builder), which is returned by the getDataRequest().
For the song count job, we want to request all songs that have been played, for every user. In order
to get all of the values written to the "info:track_plays" column, we must specify that the maximum
number of versions we want is HConstants.ALL_VERSIONS. Otherwise, we will only get the more recent
version by default.

{% highlight java %}
public KijiDataRequest getDataRequest() {
  // Retrieve all versions of info:track_plays:
  final KijiDataRequestBuilder builder = KijiDataRequest.builder();
  builder.newColumnsDef()
    .withMaxVersions(HConstants.ALL_VERSIONS)
    .add("info", "track_plays");
  return builder.build();
}
{% endhighlight %}


* setup()

* gather()


### LongSumReducer.java
<SongId, 1> -> <SongId, N> where N is how many times the song has been played.
From the KijiMR library. Passes the key through, and sums all LongWritables for a given key.


### Describe Tests
How did we test this?

### Running the Example

<div class="userinput">
{% highlight bash %}
$KIJI_HOME/bin/kiji jar \
{% endhighlight %}
</div>

#### Verify
