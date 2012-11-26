---
layout: post
title: "Pitfalls when creating a BitTorrent client"
description: ""
category: Advice
tags: [bittorrent, python, encoding, networking, tcp sockets]
---
{% include JB/setup %}

For a little background, I started out at Hacker School by learning Python, using [Learn Python the Hard Way](http://learnpythonthehardway.org/book/) and [Dive Into Python 3](http://getpython3.com/diveintopython3/), then moved on to write a BitTorrent client according to [specification](http://www.bittorrent.org/beps/bep_0003.html). The result was [BitClient](https://github.com/charmeleon/BitClient), a lightweight client that uses no third-party libraries \(not even [Twisted](http://twistedmatrix.com/trac/)!\) to torrent files. It's sufficiently far along as to be fully functional \(although it currently lacks uploading functionality\), and I would like to offer some insights on the difficulties that I faced when creating it, with the exception of architectural roadblocks since there's still plenty of room in BitClient for that. Some of the items below show my struggles with Python 3, others are about issues on how the specification communicates certain things.

### For HTTP trackers, test your GET requests on your browser

A GET request is part of HTTP, so your browser is able to parse whatever response the tracker(s) sends back. Doing this will give you an idea of what a response looks like when you fail to provide enough keys, have a malformed GET request, provide a hash that the tracker no longer serves, etc., and you will be able to do it much faster than using the terminal.

### Using sockets for GET requests is \(most likely\) overkill

BitClient currently only supports HTTP trackers, which according to the specification respond accordingly to well-documented GET requests. Initially I was connecting to these trackers using TCP sockets. I would then parse the response using regular expressions. Although it was a good exercise, it was likely a waste of my time -- I've seen GET requests plenty of times before, and I ultimately re-wrote the __tracker.py__ file in its entirety, scrapping its socket-focused approach and moved to using urllib instead. In retrospect, I got very little to show for the time that went into this. Even if you're using this project to learn about networking (which I was), there's enough interesting networking problems ahead to waste time on this.

### TypeError: 'str' does not support the buffer interface

I stumbled on this error during an early attempt at sending my GET request through a socket by way of using `my_socket.send("my_get_request")`. At the time, it was obscure enough that I had trouble comprehending the explanations on stackoverflow, even after digesting a ton of material on encoding. The issue was that `"my_get_request"`, in Python 3, is of type 'str' which is encoded as UTF-8, while the `socket.send` function only accepts 'bytes'. So an acceptable solution would be to instead write `my_socket.send(bytes("my_get_request", "UTF-8"))`. While this solves the problem, I'm still clueless as to what the 'buffer interface' is.

### TypeError: Unicode-objects must be encoded before hashing

The sister error of the aforementioned 'buffer interface', this one is spit out by `hashlib.sha1("hash this")`. It can be easily resolved by the same method as above, though check out the following paragraph for a closer look. I only mention it because this message is _much_ more human-readable than the 'buffer interface' one.

### Creating info_hash

The spec on TheoryOrg says this about the info hash:
> __info\_hash:__ urlencoded 20-byte SHA1 hash of the _value_ of the _info_ key from the Metainfo file. Note that the _value_ will be a bencoded dictionary, given the definition of the info _key_ above. 

I have strong issues with the wording here. Hashing with sha1 returns a sha1 object, and to get the hash string you can use one of two methods - `digest()` and `hexdigest()`. In Python2 both methods will return a bytes object, so you should be able to use `my_hash.hexdigest()[:20]` and think that it's right \(it's not\). I think a better phrasing for __info hash__ goes as follows \(bold emphasis on changes\):

> __info\_hash__: urlencoded 20-byte SHA1 hash __string, parsed as binary data,__ of the __bencoded _info_ key from the Metainfo file__.

I also had to battle with default encoding on this one. My bencoded strings are 'str' objects, and to hash it I need to convert to 'bytes'. Initially, I used `bytes(info_key, "UTF-8")`, but sometimes Python3 would raise a UnicodeDecodeError. This makes sense if you realize that info_key wasn't originally written to file as UTF-8, and Python was re-parsing it in such a way that it picked up unintended characters that didn't map to anything. I had to encode my bencoded string using latin-1. The line that creates my info_hash \(sans the urlencoding\), thus looks like this:
    return sha(bencoder.encode(info_key).encode("latin-1")).digest()

### Begin with simple torrents

If this is your first time building something from spec and using a language you've never used before, it's best to use simple torrents. The nuances of the spec and the language will keep you sufficiently occupied for a while, and architecturing a large, already-scaled project from scratch is not in your best interest. [Tom's test.torrent](https://github.com/charmeleon/BitClient/blob/master/torrents/test.torrent) is posibly the best torrent file to begin with. _\(As an annecdote, I should put it out here that I started with a file for Fedora 14, which had long since disappeared from the moment I picked it up. As a result, trackers either were no longer online or returned no peers, and though I was on the right path I couldn't tell because I couldn't find anyone to connect to -- so beware of dead torrents\)._

### Incorrect request size

This issue was almost entirely my fault for not reading below this line in the specification:
>> __This section is under dispute! Please use the discussion page to resolve this!__
\(In my defense, I'm of the opinion that a mandatory portion of the official specification shouldn't be listed in a section under dispute, nor hidden amongst a wall of text\). As a bonus, using an incorrect request size results in peers merely dropping you -- there's no clear indication of where you went wrong. I only realized this was an issue after following my requests on [wireshark](http://www.wireshark.org/) and noticed every peer dropping me after my request was sent. For now, I've hard-coded my request size to 2^14 \(2\*\*14 in Python\).

### Block offset

Incidentally, this is the issue that followed when I fixed the incorrect request size issue above. Taking a closer look at the specification:
>>  __request: <notextile><len=0013><id=6><index><begin><length></notextile>__
>>
>>The __request__ message is fixed length, and is used to request a block. The payload contains the following information:
>>...
>>    __begin:__ integer specifying the zero-based byte offset within the piece
>>...

I interpreted the "begin" line to mean that you enumerated the blocks. Therefore my requests came in looking like this \(severely truncated so that they fit this page\):
>>³õ^[7¯\ Ú= \(offset: 0\)
>>õ^[7¯\ Ú=Ñ \(offset: 1\)
>>^[7¯\ Ú=Ñ² \(offset: 2\)

But the right way to go about it is this: let's say that a piece is 16735 bytes and we are using 2^14-sized requests. Their offsets would be as follows:
>> Piece 1: offset 0
>> Piece 2: offset 16384
>> Piece 3: offset 32768

To clarify this section, I would suggest:
>>    __begin:__ zero-based integer specifying the offset of the data within the piece

### Incomplete/Chained/Invalid messages

So the filesharing portion of a BitTorrent client is accomplished by communicating length-prefixed messages. The more interesting issues that arise out of this occurs when you read from a socket and you receive:
* An incomplete message
* Multiple messages as a single string
* An invalid message
* Any combination of the above
With the exception of invalid messages \(upon receipt of which I would strongly advise to immediately terminate the connection with that peer\), this is one of the more interesting issues that arise from the Networking aspect of a BitTorrent client. The solution(s) is(are) left as an exercise to the reader. I merely make mention of this as a form of record to let you know such cases exist.

### Bitfields

According to TheoryOrg:
> The bitfield message is variable length, where X is the length of the bitfield. The payload is a bitfield representing the pieces that have been successfully downloaded. The high bit in the first byte corresponds to piece index 0. Bits that are cleared indicated a missing piece, and set bits indicate a valid and available piece. Spare bits at the end are set to zero. 
This description, in my opinion, doesn't do a bitfield justice. Let me represent a bitfield payload:
    payload = b'\xff\xfe\x0f'
If you check the length of the bytes above, python should tell you that it is 3. Thus, we have:
    \xff \xfe \x0f
However, if you access them individually either by index or in a `for` loop, Python will say:
    * payload[0] == 255
    * payload[1] == 254
    * payload[2] == 15
Very interesting. Note how the specification says that the high bit of the first byte corresponds to piece index 0. In the example above, I had taken this to mean that the bitfield consisted of 3 pieces, and that we only had piece 0 _since it is the only one that is truly high_. That bit of rationalizing kept me uneasy for a while until I started running into trouble -- longer torrents would not complete because my bitfield was parsed incorrectly. So here's what it means. Let's try, in Python:
    \> pcs = bin(bitfield[0])
    # pcs == '0b11111111'
    \> pcs[2:]
    '11111111'
The last line is what should be interpreted as the bitfield. In our case, it indicates that the per has the first eight pieces (pieces 0 through 7).

My suggestion:

>> The bitfield payload is a serialized string of hex values, each of which represents an integer that when converted to binary representation, indicates the status of a piece. A high byte indicates a valid and available piece, while a low byte indicates a missing piece.

### Parsing tracker's response

I was mystified by trackers' responses for almost a day. I was still getting the hang of wireshark - I could Follow a Stream, but I was always looking at the 'RAW' representation of the stream. An actual example:
    d8:completei41e12:crypto_flags16:................10:incompletei3e8:intervali1800e5:peers96:...:.S.._...2..0..[.Z...P......9z.0.M2.w..O.....D.Z...U.x..0.Ba...E....ZX.@M.0U...X......d.@r...e

It took me a while to realize that the dots were most likely characters that didn't have an ASCII representation, and wireshark was defaulting to a dot as its "I don't know what this is" flag. For half a day I was convinced that my trackers' responses were garbage. In this case, try looking at \(and properly following\) the conversation under "Hex Dump". At other times, "C Arrays" parsing was useful, especially once you work out its labeling system for peers/messages.

### If something's wrong, always look at wireshark

If something is wrong, always look at wireshark. One of the issues with building your BitTorrent client is that you're dealing with quantifiable amounts of mostly hex data. Personally, interpreting this data on sight took some time getting used to, but wireshark can make it less painful, particularly because of its aforementioned capacity to represent your packets in different encodings, which are useful at different points in this process. It can help you verify whether the endianness of your packets is in the correct order, whether peers abruptly disconnect from you at any point, and you may pick up a modicum of knowledge of some of the other Protocols that take place right under your nose!

## Fin

I hope you enjoyed the read!

