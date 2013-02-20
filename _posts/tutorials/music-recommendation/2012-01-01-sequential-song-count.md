---
layout: post
title : SequentialSongCount
categories: [tutorials, music-recommendation, 1.0.0-rc4]
tags: [music]
order : 5
description: Includes info on working with Avro
---


Instead of recommending to most popular songs to everyone using our service, we want to tailor our
recommendations based on user's listening history. For every user, we will lookup the most recent
song they have listened to and then recommend the song most frequently played after it. In order
to do that, we need to create an index so that for each song, we can quickly lookup what the
most popular songs to listened to afterwards is.

So, we need to count the number of times two songs have been played, one after another. The 
SequentialSongCount mapper and reducer allow us to do that.

### SequentialPlayCounter.java
KijiRowData -> <SongBiGram, 1> = <{firstSong : songid, secondSong : songid}, 1>

This class is very similar to our SongCount, but instead of counting the number of times a single
song has been played, we need to count the number of times two songs have been play one after the
other. So, we need our key to represent two songs played in a particular order. The easiest way
to use a complex key like this is to use [Avro](linktosomething).

* In order to count how many times two songs have been played in a row, we need to define a simple
way to 

{% highlight json %}
  /** Song play bigram. */
  record SongBiGram {
    /** The id of the first song played in a sequence. */
    string first_song_played;

    /** The id of the song played immediately after it. */
    string second_song_played;
  }
{% endhighlight %}

* implement AvroKeyWriter

* gather() (highlight setting of fields and reusing the object)


### SequentialPlayCountReducer.java
<SongBiGram, 1> -> <SongId, SongCount>
<{firstSong : songid, secondSong : songid}, 1> -> <SongId, {songId: id, count: count}>

This reducer takes in pairs of songs that have been played sequentially and the number one.
It then computes the number of times those songs have been played together, and emits the id of
the first song as the key, and a SongCount record representing the song played after the first as
the value. A SongCount record has a field containing the id of the subsequent song and a field
for the number of times it has been played after the initial song.

This reducer takes AvroKeys as input, and writes AvroKeys and AvroValues as output, so it must
implement AvroKeyReader, AvroKeyWriter, and AvroValueWriter. The keys we are emiting are just strings
so we could use a [Text](link to text key docs) key. Instead, we made the choice to use an AvroKey
so that we could use the Kiji defined [AvroKeyValue output format](link to userguide section), which
requires that you output AvroKeys and AvroValues.

Now, the schema for our avro key is so simple that we don't have to add a record to our .avdl file
in order to return the correct schema in getWriterSchema(). Instead, we can use the static methods
avro provides for creating schemas of primitive types.

{% highlight java %}
  public Schema getAvroKeyWriterSchema() throws IOException {
    // Programmatically retrieve the avro schema for a String.
    return Schema.create(Schema.Type.STRING);
  }
{% endhighlight %}

* reduce()



### Describe Tests
How did we test this?

### Running the Example

<div class="userinput">
{% highlight bash %}
$KIJI_HOME/bin/kiji jar \
{% endhighlight %}
</div>

#### Verify
