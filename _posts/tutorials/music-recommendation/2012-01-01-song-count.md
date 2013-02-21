---
layout: post
title : SongCount
categories: [tutorials, music-recommendation, 1.0.0-rc4]
tags: [music]
order : 4
description: Like WordCount, for songs.
---

### The 'Hello World!' of MapReduce
To quote Scalding Developers [Hadoop is a distributed system for counting words.](https://github.com/twitter/scalding) Unfortunately, we here at PANDORA have very few words to
count, but millions of users that play millions of different songs.

This MapReduce job uses the listening history of our users that we have stored in a Kiji table to
calculate the total number of times each song has been played. The result of this computation is
written to a text file in HDFS.  Between the map and reduce tasks described below, Hadoop performs
a group by key operation which collects all values mapped to the same key into an iterator and returns
key/iterator pairs.

<div id="accordion-container">
  <h2 class="accordion-header"> SongPlayCounter.java </h2>
     <div class="accordion-content">
{% highlight java %}
/**
 * Gatherer to count the total number of times each song has been played.
 *
 * Reads the track plays from the user table and emits (song ID, 1) pairs for each track play.
 * This gatherer should be combined with a summing reducer to count the number of plays per track.
 */
public class SongPlayCounter extends KijiGatherer<Text, LongWritable> {
  /** Only keep one Text object around to reduce the chance of a garbage collection pause.*/
  private Text mText;
  /** Only keep one LongWritable object around to reduce the chance of a garbage collection pause.*/
  private static final LongWritable ONE = new LongWritable(1);

  /** {@inheritDoc} */
  @Override
  public Class<?> getOutputKeyClass() {
    return Text.class;
  }

  /** {@inheritDoc} */
  @Override
  public Class<?> getOutputValueClass() {
    return LongWritable.class;
  }

  /** {@inheritDoc} */
  @Override
  public void setup(GathererContext<Text, LongWritable> context) throws IOException {
    super.setup(context); // Any time you override setup, call super.setup(context);
    mText = new Text();
  }

  /** {@inheritDoc} */
  @Override
  public KijiDataRequest getDataRequest() {
    // Retrieve all versions of info:track_plays:
    final KijiDataRequestBuilder builder = KijiDataRequest.builder();
    builder.newColumnsDef()
        .withMaxVersions(HConstants.ALL_VERSIONS)
        .add("info", "track_plays");
    return builder.build();
  }

  /** {@inheritDoc} */
  @Override
  public void gather(KijiRowData row, GathererContext<Text, LongWritable> context)
      throws IOException {
    NavigableMap<Long, CharSequence> trackPlays = row.getValues("info", "track_plays");
    for (CharSequence trackId : trackPlays.values()) {
      mText.set(trackId.toString());
      context.write(mText, ONE);
      mText.clear();
    }
  }

}
{% endhighlight %}
     </div>
 <h2 class="accordion-header"> LongSumReducer.java </h2>
   <div class="accordion-content">
{% highlight java %}
/**
 * <p>A WibiReducer that works of key/value pairs where the value is
 * an LongWritable.  For all integer values with the same key, the
 * LongSumReducer will output a single pair with a value equal to the
 * sum, leaving the key unchanged.</p>
 *
 * @param <K> The type of the reduce input key.
 */
public class LongSumReducer<K> extends KeyPassThroughReducer<K, LongWritable, LongWritable> {
  private LongWritable mValue;

  /** {@inheritDoc} */
  @Override
  protected void setup(Context context) {
    mValue = new LongWritable();
  }

  /** {@inheritDoc} */
  @Override
  protected void reduce(K key, Iterable<LongWritable> values,
      Context context) throws IOException, InterruptedException {
    long sum = 0;
    for (LongWritable value : values) {
      sum += value.get();
    }
    mValue.set(sum);
    context.write(key, mValue);
  }

  /** {@inheritDoc} */
  @Override
  public Class<?> getOutputValueClass() {
    return LongWritable.class;
  }
}
{% endhighlight %}
    </div>
</div>

### SongPlayCounter.java
The SongPlayCounter is an example of a [Gatherer](link-to-gatherer-docs), which is essentially a
mapper that gets input from a KijiTable.  SongPlayCounter proceeds through discreet stages:
* Setup reusable resources.
* Read all values from column: "info:track_plays".
* Process the data from "info:track_plays" and emit a key/value pair for each track id each time
  it occurs.

#### Initialize Resources
  Prepares any resources that may be needed by the gatherer.  In Hadoop, reusable objects are
  commonly instantiated only once to protect against possible scanner timeout exceptions caused
  by garbage collection.  Because setup() is an overriden method, we call super.setup() to
  ensure that all resources are initialized properly.  If you open resources in setup(), be
  sure to close them in the corresponding cleanup() method.

{% highlight java %}
  public void setup(GathererContext<Text, LongWritable> context) throws IOException {
    super.setup(context); // Any time you override setup, call super.setup(context);
    mText = new Text();
  }
{% endhighlight %}

#### Read track play data from the table
A gatherer takes input from a table, so it must declare what data it will need. It does this in the
form of a [KijiDataRequest](link-data-request-builder), which is returned by the getDataRequest().
For the song count job, we want to request all songs that have been played, for every user. In order
to get all of the values written to the "info:track_plays" column, we must specify that the maximum
number of versions we want is HConstants.ALL_VERSIONS. Otherwise, we will only get the most recent
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

#### Process track play data into key/value pairs for occurrences
  Called once for each row in the Kiji table, gather() uses a GathererContext to write
  key/value pairs for each track id paired with one (1) as a tally.

{% highlight java %}
  public void gather(KijiRowData row, GathererContext<Text, LongWritable> context)
      throws IOException {
    NavigableMap<Long, CharSequence> trackPlays = row.getValues("info", "track_plays");
    for (CharSequence trackId : trackPlays.values()) {
      mText.set(trackId.toString());
      context.write(mText, ONE);
      mText.clear();
    }
  }
{% endhighlight %}

### LongSumReducer.java
The LongSumReducer calls reduce() on each key/iterator pair from the GroupByKey phase to
produce a total play count for each track id.  Because summing is a common MapReduce operation,
LongSumReducer.java is provided by the KijiMR library.  LongSumReducer has two simple stages:
* Setup reusable resources.
* Reduce key/iterator pairs to key/value pairs and write them to the output collector.

#### Initialize Resources
Prepares reusable resources to avoid timeout exceptions caused by garbage collection.

{% highlight java %}
  protected void setup(Context context) {
    mValue = new LongWritable();
  }
{% endhighlight %}

#### Sum play counts for each song
  Called for each key/iterator pair from the GroupByKey phase above, reduce() combines the
  results of each iterator to singular values by summing the tallies and writes the resulting
  value for each key to the output collector.

{% highlight java %}
  public void reduce(K key, Iterator<LongWritable> values,
                     OutputCollector<K, LongWritable> output,
                     Reporter reporter)
    throws IOException {

    // sum all values for this key
    long sum = 0;
    while (values.hasNext()) {
      sum += values.next().get();
    }

    // output sum
    output.collect(key, new LongWritable(sum));
  }
{% endhighlight %}

### Describe Tests
How did we test this?

### Running the Example

<div class="userinput">
{% highlight bash %}
$ kiji gather \
      --gatherer=org.kiji.examples.music.gather.SongPlayCounter \
      --reducer=org.kiji.mapreduce.lib.reduce.LongSumReducer \
      --input="format=kiji file=${HDFS_ROOT}/users" \
      --output="format=text file=${HDFS_ROOT}/output.txt_file nsplits=2" \
      --lib=${LIBS_DIR}
{% endhighlight %}
</div>

#### Verify

To confirm that the gather job worked:

<div class="userinput">
{% highlight bash %}
$ hadoop fs -text ${HDFS_ROOT}/output.text_file/part-r-00000
song-1  100
song-10 272
song-12 101
...
{% endhighlight %}
</div>

Your output may vary because ordering is not guaranteed after Hadoop reduce jobs,
but the format should be the same.
