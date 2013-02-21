---
layout: post
title : NextTopSong
categories: [tutorials, music-recommendation, 1.0.0-rc4]
tags: [music]
order : 6
description: Outputing to a Kiji table in a MapReduce job.
---

<div id="accordion-container"> 
  <h2 class="accordion-header"> IdentityMapper.java </h2> 
     <div class="accordion-content"> 
{% highlight java %}
public class IdentityMapper
    extends KijiMapper<
        AvroKey<CharSequence>, AvroValue<SongCount>, AvroKey<CharSequence>, AvroValue<SongCount>>
    implements AvroKeyWriter, AvroValueWriter, AvroKeyReader, AvroValueReader {

  /** {@inheritDoc} */
  @Override
  public Class<?> getOutputKeyClass() {
    return AvroKey.class;
  }

  /** {@inheritDoc} */
  @Override
  public Class<?> getOutputValueClass() {
    return AvroValue.class;
  }

   /** {@inheritDoc} */
  @Override
  public Schema getAvroValueReaderSchema() throws IOException {
    return SongCount.SCHEMA$;
  }

  /** {@inheritDoc} */
  @Override
  public Schema getAvroKeyReaderSchema() throws IOException {
    return Schema.create(Schema.Type.STRING);
  }

  /** {@inheritDoc} */
  @Override
  public Schema getAvroValueWriterSchema() throws IOException {
    return SongCount.SCHEMA$;
  }

  /** {@inheritDoc} */
  @Override
  public Schema getAvroKeyWriterSchema() throws IOException {
    return Schema.create(Schema.Type.STRING);
  }

  /** {@inheritDoc} */
  @Override
  public void map(AvroKey<CharSequence> key, AvroValue<SongCount> value, Context context)
    throws IOException, InterruptedException {
    context.write(key, value);
  }
}
{% endhighlight %}
     </div> 
 <h2 class="accordion-header"> TopNextSongsReducer.java </h2> 
   <div class="accordion-content"> 
{% highlight java %}
public class TopNextSongsReducer
    extends KijiTableReducer<AvroKey<CharSequence>, AvroValue<SongCount>>
    implements AvroKeyReader, AvroValueReader {
    private static final Logger LOG = LoggerFactory.getLogger(TopNextSongsReducer.class);

  /** An ordered set used to track the most popular songs played after the song being processed. */
  private TreeSet<SongCount> mTopNextSongs;

  /** The number of most popular next songs to keep track of for each song. */
  private final int mNumberOfTopSongs = 3;

  /** A list of SongCounts corresponding to the most popular next songs for each key/song. */
  private TopSongs mTopSongs;

  /** {@inheritDoc} */
  @Override
  public void setup(Context context) throws IOException, InterruptedException {
    super.setup(context); // Any time you override setup, call super.setup(context);
    mTopSongs = new TopSongs();
    // This TreeSet will keep track of the "largest" SongCount objects seen so far. Two SongCount
    // objects, song1 and song2, can be compared and the object with the largest value in the field
    // count will the declared the largest object.
    mTopNextSongs = new TreeSet<SongCount>(new Comparator<SongCount>() {
      @Override
      public int compare(SongCount song1, SongCount song2) {
        if (song1.getCount().compareTo(song2.getCount()) == 0) {
          return song1.getSongId().toString().compareTo(song2.getSongId().toString());
        } else {
          return song1.getCount().compareTo(song2.getCount());
        }
      }
    });
  }

  /** {@inheritDoc} */
  @Override
  protected void reduce(AvroKey<CharSequence> key, Iterable<AvroValue<SongCount>> values,
      KijiTableContext context) throws IOException {
    // We are reusing objects, so we should make sure they are cleared for each new key.
    mTopNextSongs.clear();

    // Iterate through the song counts and track the top ${mNumberOfTopSongs} counts.
    for (AvroValue<SongCount> value : values) {
      // Remove AvroValue wrapper.
      SongCount currentSongCount = SongCount.newBuilder(value.datum()).build();

      mTopNextSongs.add(currentSongCount);
      // If we now have too many elements, remove the element with the smallest count.
      if (mTopNextSongs.size() > mNumberOfTopSongs) {
        mTopNextSongs.pollFirst();
      }
    }
    // Set the field of mTopSongs to be a list of SongCounts corresponding to the top songs played
    // next for this key/song.
    mTopSongs.setTopSongs(Lists.newArrayList(mTopNextSongs));
    // Write this to the song table.
    context.put(context.getEntityId(key.datum().toString()), "info", "top_next_songs", mTopSongs);
  }

  /** {@inheritDoc} */
  @Override
  public Schema getAvroValueReaderSchema() throws IOException {
    return SongCount.SCHEMA$;
  }

  /** {@inheritDoc} */
  @Override
  public Schema getAvroKeyReaderSchema() throws IOException {
    return Schema.create(Schema.Type.STRING);
  }
}
{% endhighlight %}
    </div> 
</div>

### IdentityMapper.java


### TopNextSongsReducer.java 
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
kiji mapreduce \
--mapper=org.kiji.examples.music.map.IdentityMapper \
--reducer=org.kiji.examples.music.reduce.TopNextSongsReducer \
--input="format=avrokv file=${HDFS_ROOT}/output.sequentialPlayCount" \
--output="format=kiji table=${KIJI}/songs nsplits=1" \
--lib=${LIBS_DIR}
{% endhighlight %}
</div>

#### Verify

<div class="userinput">
{% highlight bash %}
kiji ls --kiji=${KIJI}/songs --columns=info:top_next_songs --max-rows=3
{% endhighlight %}
</div>
