# wsrep API node application

## Overview

This is a simple application to demonstrate the usage of wsrep API. It
deliberately does nothing useful in order to present as concentrated and
concise API usage as possible.

The program is deliberately written in C to demonstrate the naked API usage.
For C++ example see a much more advanced integration library at
https://github.com/codership/wsrep-lib

## High level architecture

Process-wise the program consists of an endless main loop that periodically
samples and prints performance stats and a configurable number of "master" and
"slave" threads, with master threads loop executing "transactions" and
replicating resulting "write sets" and slave threads receiving and processing
the write sets from other nodes.

Object-wise the program is composed of two main objects: `store` and `wsrep`.
'store' object contains application "state" and generates and commits changes
to the state. `wsrep` object contains cluster context and provides interface
to it. Changes generated by `store` are replicated and certified through
`wsrep` and then committed to `store`.

## Unit descriptions (in alphabetical order)

#### ctx.h
A small header to declare the application context structure.

#### log.*
Implements logging functionality for the application AND
**a logging callback** for the wsrep provider.

#### main.c
Defines `main()` routine that initializes storage and wsrep provider, starts
the worker threads and loops in a statistics collection loop. Even though it is
not designed to return it still shows the deinitialization order.

#### options.*
Implements reading configuration options from the command line, does not have
anything related to wsrep API, but shows which additional parameters must be
configured for the program to make use of wsrep clustering.

#### socket.*
Network sockets boilerplate code for setting TCP connections between processes
(for SST). Has nothing wsrep-related and can be ignored.

#### sst.*
Defines **SST callbacks** for the wsrep provider and shows how to asynchronously
implement state snapshot transfer (yes, you don't want to spend eternity in
callbacks).

#### stats.*
Implements performance stats collecting function for the main loop. While it is
an absolutely optional provider functionality, still it shows how to use that.

#### store.*
Defines the `store` object that pretends to store and modify some data in a
"transactional" manner. It provides the caller that intends to do a change with
a *change data* and a *key* for replication and certification.

#### trx.*
Defines routines to process local and replicated transactions.

#### worker.*
Implements worker thread pool functinality. Worker threads run routines defined
in 'trx.*'. Also implements **apply callback** for the wsrep provider.

#### wsrep.*
Maintains wsrep cluster context: provider instance and cluster membership view.
While there is little use for the latter in this primitive application, still
it shows **connected and view callbacks** usage. But mostly, for this
application its purpose is to initialize the provider, connect to the cluster
and offer access to initialized provider for other parts of the program.

## Example usage
```
./node -f /tmp/galera/0 -v /tmp/galera/0/galera/lib/libgalera_smm.so -o 'pc.weight=2;evs.send_window=2;evs.user_send_window=1;gcache.recover=no' -s 8 -m 16
```