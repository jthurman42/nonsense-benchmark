## Yet another shootout

So you want to see how well your programming language of choice deals with
highly concurrent network serving, and you want it to be under controlled
circumstances. This repository contains the beginnings of a way to compare
how different languages scale under a specific kind of load.

Right now there's a simple reference server, a multi-CPU clustered server, and
a highly concurrent benchmarking client. All are written in Node. To get the
server working, cd into `node/clients` and run `npm install`, assuming you have
Node installed.

The intention of this repo is that other implementations will be added. Send me
pull requests to add yours.

### The workload

The reference server is a very lightweight version of a proof-of-work server.
You provide it with a string, and it returns that string and a nonce. Depending
on how the server is set up (which is hardcoded right now), the string and the
nonce, when concatenated and fed into OpenSSL's SHA256 implementation, will
produce a hex SHA with the last x digits being 0 (where x is hardcoded,
currently at 2 -- it's important that the client and server match in this
regard).

Since the goal of this benchmark is to provide just enough CPU load to keep it
from being a test of how quickly you can exhaust the available file descriptors
and / or sockets on the server's host, the workload should stay fairly light,
in the 10-30ms range.

### Caveats

The client should not trust the server:

* It is up to the client to verify that the string returned by the work server
  is the same string that the client originally sent.
* The client must prove that the work was done itself by verifying that the
  string + nonce combo produce the required result.
* No timing information is generated by the server; this means that the timing
  and throughput numbers returned by the client include network overhead, but
  it's hard to think of a real-world network service where this wouldn't be the
  case.

In addition, it's worth pointing out that the whole point of a proof-of-work
service is to be CPU-intensive, and at least in the case of Node, this
benchmark tests the speed of context-switching between C++ and JavaScript as
well as the speed of the OpenSSL SHA256 implementation. I may try swapping in a
pure JS SHA256 implementation just for lulz at some point.

### TODO

* Tweak client performance. You know, with profiling and stuff.
* Build a distributed client-server version of the benchmarker so that we can
  hit a service from multiple hosts simultaneously and better simulate a
  realistic, intensive workload.
* Add more implementations of the servers and clients.
