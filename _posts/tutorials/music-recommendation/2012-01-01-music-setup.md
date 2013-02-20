---
layout: post
title: Setup
categories: [tutorials, music-recommendation, 1.0.0-rc4]
tags: [music]
order: 2
description: Setup and compile instructions for the music recommendation tutorial.
---

### Setup Hadoop Environment
Start bento. explain more


### Install Kiji and Create Tables

Install your Kiji instance:

<div class="userinput">
{% highlight bash %} $
kiji install --kiji=${{ "{KIJI" }} }
{% endhighlight %}
</div>

Create the Kiji music tables:

<div class="userinput">
{% highlight bash %}
kiji-schema-shell --kiji=$KIJI --file=music_schema.ddl
{% endhighlight %}
</div>


### Get Data

Use the python script to generate data sets:

<div class="userinput">
{% highlight bash %}
./data_generator.py --output-dir=data/
{% endhighlight %}
</div>

 *we should have an alternative for people who dont want/have python.*
This should generate 3 JSON files: data/song-dist.json, data/song-metadata.json and data/song-plays.json.

Upload the data set to HDFS:

<div class="userinput">
{% highlight bash %}
hadoop fs -mkdir $HDFS_ROOT/kiji-mr-tutorial
hadoop fs -copyFromLocal data/*.json $HDFS_ROOT/kiji-mr-tutorial/
{% endhighlight %}
</div>

