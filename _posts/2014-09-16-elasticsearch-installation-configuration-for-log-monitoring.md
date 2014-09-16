---
layout: post
title: ElasticSearch installation and configuration for Log monitoring
comments: true
---

## Starting Point

This post has been done starting on a server with a Ubuntu 14.04 fresh install.

## Step 1 : Download ELK stack

{% highlight bash %}
sudo apt-get install unzip

mkdir Download
cd Download
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.3.2.zip

unzip elasticsearch-1.3.2.zip
{% endhighlight %}

## Step 2 : Install JDK 7+

2 possibilities : OpenJDK or Oracle JDK

### 1st choice : OpenJDK

Example is given for Java 7. Replace 7 by 8 for OpenJDK 8.

{% highlight bash %}
sudo apt-get install openjdk-7-jdk
{% endhighlight %}

### 2nd choice : Oracle JDK

Example is given for Oracle Java 8.

{% highlight bash %}
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
{% endhighlight %}

### Verification

{% highlight bash %}
    java -version
{% endhighlight %}
Should return the correct JDK (OpenJDK or Oracle) and the correct version (7 or 8).

## Step 3 : Add a dedicated user for Elasticsearch

Create a user for Elasticsearch (named *elasticsearch*) and prepare a folder for the binaries.
{% highlight bash %}
sudo adduser elasticsearch
su elasticsearch
cd ~
{% endhighlight %}

Create 2 folders : *elasticsearch-1.3.2* to handle binaries and *software* for symbolic links (useful when upgrade will happen)
{% highlight bash %}
mkdir software
mkdir elasticsearch-1.3.2
exit
{% endhighlight %}

Copy binaries from Download folder to *elasticsearch* folder.
Do not forget to set proper permissions and owner on Elasticsearch binaries
{% highlight bash %}
sudo cp -R elasticsearch-1.3.2 /home/elasticsearch/software/elasticsearch-1.3.2
sudo chown elasticsearch:elasticsearch -R /home/elasticsearch/software/
{% endhighlight %}

Create symbolic link in *software*, set permissions and create working folders for Elasticsearch (*data*, *logs*, *work*)
{% highlight bash %}
su elasticsearch
cd ~/software
ln -s elasticsearch-1.3.2 elasticsearch
chmod 770 -R *
mkdir ~/data
mkdir ~/logs
mkdir ~/work
{% endhighlight %}

## Step 4 : Configure Elasticsearch

Edit Elasticsearch configuration file (as *elasticsearch* user)
{% highlight bash %}
cd ~/software/elasticsearch/config
vim elasticsearch.yml
{% endhighlight %}

Elasticsearch is configured by default for a Search usage : few indexing requests and lot of search requests.
In the case of Log monitoring using Logstash / Kibana, it's the opposite : lot of indexing requests and few search requests.

My recommandation is to change the dedicated thread pools in Elasticsearch.
We will setup the *search* thread pool size to a fixed value of 20, the *bulk* thread pool (used to laod data into ES) size to 60 and the *index* thread pool size to 20

Edit or add the following properties :
{% highlight yaml %}
# Disable dynamic scripting for security reasons (see http://www.elasticsearch.org/blog/scripting-security/)
script.disable_dynamic: true

node.master: true

node.data: true

# In my case, I only need 2 shards. Configure as needed for your use case
index.number_of_shards: 2

# In my case, I only need 0 replica. Configure as needed for your use case
index.number_of_replicas: 0

# ES will store all its data in this folder
path.data: /home/elasticsearch/data

# ES will store all its working stuff in this folder
path.work: /home/elasticsearch/work

# ES will store all its logs in this folder
path.logs: /home/elasticsearch/logs

bootstrap.mlockall: true

# disable multicast discovery
discovery.zen.ping.multicast.enabled: false

## Threadpool Settings tuned for Log monitoring activity ##
# Search pool
threadpool.search.type: fixed
threadpool.search.size: 20
threadpool.search.queue_size: 100

# Bulk pool
threadpool.bulk.type: fixed
threadpool.bulk.size: 60
threadpool.bulk.queue_size: 300

# Index pool
threadpool.index.type: fixed
threadpool.index.size: 20
threadpool.index.queue_size: 100

# Indices settings
indices.memory.index_buffer_size: 30%
indices.memory.min_shard_index_buffer_size: 12mb
indices.memory.min_index_buffer_size: 96mb

# Cache Sizes
indices.fielddata.cache.size: 15%
indices.fielddata.cache.expire: 6h
indices.cache.filter.size: 15%
indices.cache.filter.expire: 6h

# Indexing Settings for Writes
# I setup a higher flush threshold to many flushes.
# Value should be determined according to the rate of incoming documents (~50 docs/seconds in my use case)
index.refresh_interval: 30s
index.translog.flush_threshold_ops: 50000
{% endhighlight %}

## Step 5 : Adjust Elasticsearch memory settings

Connect as *elasticsearch* user :
{% highlight bash %}
vim ~/.bashrc

export ES_HEAP_SIZE=4G
{% endhighlight %}

## Step 6 : check if Elasticsearch is working

Connect as *elasticsearch* user :
{% highlight bash %}
cd ~/software/elasticsearch/bin
./elasticsearch
{% endhighlight %}

**started** should appear in the console.

Kill the server with Ctrl-C.

## Step 7 : Fine tune OS settings

Connect as a sudoable user :
{% highlight bash %}
sudo sysctl -w vm.max_map_count=262144

sudo vim /etc/sysctl.conf
# Change (or add) the following value to make max_map_count permanent :
vm.max_map_count = 262144
{% endhighlight %}

## Step 8 : start Elasticsearch as a daemon

Connect as *elasticsearch* user :
{% highlight bash %}
cd ~/software/elasticsearch/bin
./elasticsearch -d
{% endhighlight %}



{% include twitter.html %}