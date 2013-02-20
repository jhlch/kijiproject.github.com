---
layout: post
title : Music Recommendation Producer
categories: [tutorials, music-recommendation, 1.0.0-rc4]
tags: [music]
order : 7
description: Read and write to the same row of a table.
---


### NextSongRecommender.java
The NextSongRecommender is an example of a [KijiProducer](link-to-userguide). A producer operates on
a single row of input data and generates new outputs that are written to the same row. This producer
will use the songId of the most recent song played for a user (represented by a row) to lookup
a list of the top next songs, and then generate a recommendation from that list.

* KeyValueStores allow you to access external data sources in a MapReduce job. In this case, we will
use the "top_next_songs" column of our songs table as a KeyValueStore. In order to access KeyValueStores
in a MR job, the class that needs the external data must implement KeyValueStoreClient. This
interface requires that you declare the names of your KeyValueStores as well as a default
implementation.

### Describe Tests

Here it is important to mention how we overrode keyvaluestore bindings, and how this is the correct
example of doing this programmatically. When we run the job, we will override the bindings using
an xml file.

### Running the Example

<div class="userinput">
{% highlight bash %}
$KIJI_HOME/bin/kiji jar \
{% endhighlight %}
</div>

#### Verify
They should just look at the number of records in the mapper and out the reducer. Maybe even direct
them to the webUI for the jobtracker? They can look at the table after the second half of the job is
done.
