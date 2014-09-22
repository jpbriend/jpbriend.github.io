---
layout: post
title: Connecting to a SFTP server in Java
comments: true
---

Working in a financial environment, a lot of transfers are done using different ways.

One of these way is file transfer, usually done using FTP protocol. However FTP protocol is absolutely not secured (data and password are in clear over the network).

The operational choice is often SFTP.
SFTP (SSH File Transfer Protocol) is not FTP over SSH, but a rather new protocol, designed since the beginning by working group IETF SECSH.
SFTP is a communication protocol running above SSH, do not confuse with File Transfer Protocol over SSL, FTPS.

The goal of this post is to write a piece of code to connect to a SFTP server, upload a file and dlownload another file.
We will of course write a unit test and for this task, we will use an embedded SFTP server.

## The client : Jsch

[Jsch](http://www.jcraft.com/jsch/) is a 100% Java implementation of [SSH2](http://ietf.org/html.charters/secsh-charter.html).

This library will allow us to communicate with a SFTP server.
Authentication with this kind of server may done done in 2 ways (do not forget SFTP is SSH, so we will find common stuff) :

* login and password
* login and private key (whose public key has been previously registered on the server)

In this article, we will use login/password authentication (because it's simpler to implement).

## A SFTP server for local development

For coding, I advise you to use a little SFTP server, executed on your workstation :
[miniSFTPServer](https://github.com/jpbriend/sftp-example/blob/master/src/main/resources/miniSFTPserver.exe) is a very light simple SFTP server, allowing you to work locally.

You just have to launch it, then configure a lort, a login, a password and a root path.

## SFTP connection

SFTP server is now up & running : let's start coding !

Connecting to the server is quite easy :

{% highlight java %}
jsch = new JSch();
session = jsch.getSession(login, server, port);

// Java 6 version
session.setPassword(password.getBytes(Charset.forName("ISO-8859-1")));

// Java 5 version
// session.setPassword(password.getBytes("ISO-8859-1"));

Properties config = new java.util.Properties();
config.put("StrictHostKeyChecking", "no");
session.setConfig(config);

session.connect();
{% endhighlight %}

Then, you have to add a <em>com.jcraft.jsch.Channel</em> of type "sftp" (which is a <em>com.jcraft.jsch.ChannelSftp</em> once the channel is accepted) :

{% highlight java %}
// Initializing a channel
channel = session.openChannel("sftp");
channel.connect();
c = (ChannelSftp) channel;
{% endhighlight %}

## Uploading a file

This is quite simple again : the previously created <em>Channel</em> gives us a method <em>put</em> :

{% highlight java %}
c.put(sourceFile, destinationFile);
{% endhighlight %}

This code is using the 2-parameters version. This method exists in a lot of versions, check them to find the best one for your use case.

## File downloading

Same player shoot again : the same <em>Channel</em> has a <em>get</em> method :

{% highlight java %}
c.get(sourceFile, destinationFile);
{% endhighlight %}

## Disconnection

As every Java stream, it's important to close the <em>Channel</em>, then the <em>Session</em> in order to disconnect properly !

## Unit testing : Mina SSHD


[Mina](http://mina.apache.org/) project is offering a very interesting sub-module for unit testing SFTP servers : [SSHD](http://mina.apache.org/sshd/).

We will use SSHD to start an in-memory SFTP server which will enable us to test our SFTP client.


### @Before

In a <em>setUp</em> method (annotated with <em>@Before</em>), we will start a <em>org.apache.sshd.SshServer</em>

{% highlight java %}
@Before
public void setUp() throws IOException {
  // Init sftp server stuff
  sshd = SshServer.setUpDefaultServer();
  sshd.setPort(port);
  sshd.setPasswordAuthenticator(new MyPasswordAuthenticator());
  sshd.setPublickeyAuthenticator(new MyPublickeyAuthenticator());
  sshd.setKeyPairProvider(new SimpleGeneratorHostKeyProvider());
  sshd.setSubsystemFactories(Arrays.<NamedFactory<Command>>asList(new SftpSubsystem.Factory()));
  sshd.setCommandFactory(new ScpCommandFactory());

  sshd.start();
  ...
  ...
}
{% endhighlight %}

Method <em>setUpDefaultServer()</em> automatically preconfigures the <em>SshServer</em>.
You just have to add the listening port, implement a very simple <em>PasswordAuthenticator</em> and a ,em>PublicKeyAuthenticator</em>.

Hard part is the next one : adding a <em>ChannekSftp</em> is not easy.

{% highlight java %}
sshd.setSubsystemFactories(Arrays.<NamedFactory<Command>>asList(new SftpSubsystem.Factory()));
{% endhighlight %}

Once server is started, the unit test is very easy to write.

### @After

Do not forget to shutdown the <em>SshServer</em> at the end of the unit tests execution :

{% highlight java %}
sshd.stop();
{% endhighlight %}

## Source Code

You can get all this code with comments and logs in a Maven project :

[https://github.com/jpbriend/sftp-example](https://github.com/jpbriend/sftp-example)


{% include twitter.html %}