---
layout: post
title: ELK stack - Using Collectd for System monitoring
comments: true
---

## Context

ELK stack (ElasticSearch, Logstash and Kibana) is often used for applicative monitoring, based on applications logs.
However it would be useful to also aggregate System metrics in Kibana dashboards.
I will present you a solution to add system (CPU, Memory, disk, network) indicators into your ELK stack.
I will also add Oracle monitoring using the [Collectd Oracle plugin](https://collectd.org/wiki/index.php/Plugin:Oracle).

## Collectd

[Collectd](http://www.collectd.org) is a daemon which polls at regular interval (10s by default) metrics where it is run.
This data may be send to any external system (as long as a plugin is available), a Logstash daemon in our case.

## Build and install Collectd :

In this post, I will show you how to build Collect in order to use the Oracle plugin.

### Download and untar

First step is simple : grab the Collectd sources from the official web site : [http://www.collectd.org/download.shtml](http://www.collectd.org/download.shtml).

Unzip the sources on your disk :

{% highlight bash %}
tar xzf collect-5.4.1.tar.gz
{% endhighlight %}

### Set Oracle environment variables

2 environments variables must be set before building the Oracle plugin : **ORACLE_HOME** and **LD\_LIBRARY\_PATH** :

{% highlight bash %}
export ORACLE_HOME=/path/to/oracle/installation
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
{% endhighlight %}

### Launch configure step

Once these 2 environment variables are set, we can configure the Build process to build Collectd, the Oracle plugin and install them in a specific directory (*/path/to/my/custom/installation*):

{% highlight bash %}
cd collect-5.4.2.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
./configure --prefix=/path/to/my/custom/installation --enable-oracle
{% endhighlight %}

Check that Oracle module and **folder** are reported as *yes* by the Configure step :

{% highlight bash %}
...
Libraries :
...
oracle . . . . . yes
...
Modules :
...
oracle . . . . . yes
...
{% endhighlight %}

### make all install

If previous check is ok, then we can build and install Collectd and its plugins :
{% highlight bash %}
make all install
{% endhighlight %}

Collectd and Oracle (plus others) plugins are now installed in */path/to/my/custom/installation* path.

### Configure Collectd

Collect configuration is done in the file *collect.conf* located in *etc* folder.

Let's edit the file and configure it this way :

{% highlight xml %}
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{% endhighlight %}

Outputing data to Logstash is done by the network plugin : we specify the Logstash IP and port.

### Launching Collectd

Collectd will run as a daemon.

The script is located in the *sbin* directory :

{% highlight bash %}
cd /path/to/my/custom/installation
cd sbin
./collect -C ../etc/collectd.conf
{% endhighlight %}

## Logstash configuration

Logstash's configuration to receive Collectd events is quite simple : a collectd output plugin exists.

Just add this input plugin to your Logstash configuration :

{% highlight yaml %}
input {
    ...
    collectd {}
    ...
}
{% endhighlight %}

Relaunch Logstash and events should begin to appear in your ElasticSearch clsuter :)

It's time for you to play with Kibana !
Have fun.

{% include twitter.html %}