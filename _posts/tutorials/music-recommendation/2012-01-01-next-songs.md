---
layout: post
title : NextTopSong
categories: [tutorials, music-recommendation, 1.0.0-rc4]
tags: [music]
order : 6
description: Outputing to a Kiji table in a MapReduce job.
---

Short explanation of what the mapper and reducer do.

### Mapper.java
The re-keying of our records has already happened. this mapper is the identity. Explain that.

### Reducer.java 
* explain the logic of how we get a list of the top songs by using a tree set with a special comparator.
We only really want ordering for the counts, but we must include the song id in the comparator,
otherwise the equals comparison that happens will not add songs with the same numbers of counts
to the *set*.

* explain that we want to write to a table so that we can look up the top next songs later. Thus,
implement KijiTableReducer. The context.put + its expected params should be explained.

### Describe Tests
Talk about how cool it is that we can test the result of a sequence of jobs in a unit test.
Explain verifying the output to tables using a table reader.

### Running the Example

<div class="userinput">
{% highlight bash %}
$KIJI_HOME/bin/kiji jar \
{% endhighlight %}
</div>

#### Verify

ls the table
