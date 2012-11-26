---
layout: post
title: "A reactor with Python's asyncore library"
description: ""
category: tutorial
tags: []
---
{% include JB/setup %}
## Asyncore Overview

Python's asyncore module is a \(very\) light library that facilitates async socket IO \(I've read that it can also handle file IO, but I have not explored that \functionality). \[At this point I should mention that this will be a very long post, due to the fact that I will be debugging several steps\]

### ...but Twisted does reactors for you

It is true that Twisted does reactors very well! In fact, a websearch for "python reactor" returns a large set of results, a majority of which are specific to Twisted, but do you understand how they work? Of course, if understanding such a thing is not interesting to you, then do not read this, as you will feel your time to be better spent elsewhere. \[Also, as of this writing, the Twisted library does not offer support for Python 3\]

## What is a Reactor and why should you care?

Wikipedia's page for reactor says: "The reactor design pattern is an event handling pattern for handling service requests delivered concurrently to a service handler by one or more inputs."

That's a good enough explanation, but I'd rather explain it by describing a concrete example. Let's say you'd like to build a chat server. For simplicity, let's build a one-sided chat server - clients can connect and send messages to the server, and the server will send back a notification that the message was received.

First, let's write this really basic server:

{% highlight python linenos %}
import asyncore

class Server(asyncore.dispatcher):
  def __init__(self, host, port):
    asyncore.dispatcher.__init__(self)
    self.create_socket()
    self.set_reuse_addr()
    self.bind((host, port))
    self.h = []
    self.listen(5)  # max number of connections to queue
  
  def handle_accepted(self, sock, addr):
    print("Incoming connection from %s:%s" % sock.getpeername())
    self.h.append(Handler(sock))

class Handler(asyncore.dispatcher):
  def __init__(self, sock):
    asyncore.dispatcher.__init__(self, sock)
    self.writebuffer = b""
  
  def handle_read(self):
    data = self.recv(8192)
    if data:
      print("[client] : %s" % data)
      self.writebuffer = b"Message received"
  
  def handle_close(self):
    print("Client has left the building")
    self.close()
  
  def handle_write(self):
    if self.writebuffer:
      self.send(self.writebuffer)
      self.writebuffer = b""

server = Server('localhost', 8080)
asyncore.loop()
{% endhighlight %}

Running this piece of code will only give you a blank screen, so let's hold off on that until we write the client. But before that let's quickly go through the server code.

First, note that there are no `select()` calls, no socket map, no socket blocking or loop declaration in the entirety of our Server code. The fact is that asyncore abstracts all of that, and as an object that subclasses one of asyncore's classes is created, it gets mapped by asyncore and as long as it lives it will be handled by the `asyncore.loop()` call \[more on that later\].

Let's also note that by subclassing `asyncore.dispatcher`, we have several methods that we can choose to override. The ones I believe important are:
{% highlight python %}
handle_read()
handle_write()
handle_connect()
handle_close()
handle_accepted()
{% endhighlight %}
You should also see that not all of these are overwritten by both classes

Additionally, subclassing asyncore gives us access to socket calls as though they were native. \(Also, note that an \_\_init\_\_ method can be omitted for Handler -- the only reason I used it was the need to declare the writebuffer variable, which is crucial\). In any case, the full range of socket calls is available to us natively - e.g.:
{% highlight python %}
connect(address) -> self.connect(address)
send(data) -> self.send(data)
recv(data) -> self.recv(data)
[...]
{% endhighlight %}
\[technically you can override these as well, but I'd advise against it for no reason other than it doesn't seem to be a particularly good one\]

Now let's type out the client.

{% highlight python linenos %}
  def handle_write(self):
    if self.writebuffer:
      self.send(self.writebuffer)
      self.writebuffer = b""
    else:
      print("No write buffer")

  def add_to_buffer(self, message):
    if isinstance(message, str):
      message = bytes(message, "UTF-8")
    message = b''.join(a for a in [self.clientid, message])
    self.writebuffer = message

  def send_all(self):
    if self.writebuffer:
      self.send(self.writebuffer)
      self.writebufer = b""

if __name__ == "__main__":
  c = Client()
  msgsent = False
  t = 0
  while True:
    try:
      msg = input("> ")
      if msg:
        c.add_to_buffer(msg)
        c.send_all()
    except KeyboardInterrupt:
      print("Exiting client")
      sys.exit(0)
asyncore.loop()
{% endhighlight %}
