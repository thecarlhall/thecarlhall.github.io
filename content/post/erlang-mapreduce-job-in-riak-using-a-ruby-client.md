---
title: "Erlang Map/Reduce Job in Riak using a Ruby Client"
date: "2013-02-11"
categories: 
  - "development"
tags: 
  - "erlang"
  - "map-reduce"
  - "riak"
  - "ruby"
---

_Note: I am horrible at Erlang, but have figured out enough to construct a Map/Reduce job. Hopefully these notes serve as more than a warning._

It's rarely possible to know every way you will want to access your data. Riak has secondary indices (2i) but if you don't have one that represents what you want to query, it can be time consuming to populate one of these when you have a lot of documents. Ad-hoc queries are rarely where a database will shine, but when you have a one-off job, sometimes Map/Reduce is the only option you have.

<!--more-->

In the example code below, I'm using the [Riak Ruby Client](https://github.com/basho/riak-ruby-client) to talk to a Riak ring. I'm getting my client from [Ripple](https://github.com/basho/ripple) because it is already setup by the time this method is called. Create a client in whatever way makes sense for your application. Look below the code example for some line-specific notes.

{{< gist thecarlhall 4751334 >}}

### Erlang Notes

- **lines 17, 38** - Define the language of the `map` and `reduce` phases, respectively. Yes, this means you can use a different language in the `map` phase than you do in the `reduce` phase (e.g. `map: erlang`, `reduce: javascript`).

- **lines 19-35, 40-42** - This is the real meat of the processing puzzle. When defining Erlang functions here, you have to use anonymous functions which start with `fun`.

- **line 19** - Defining these functions requires some knowledge of what's to be passed in, so let's look at what the arguments are
    - **Obj** - Riak object/document as retrieved from the bucket.
    
    - **_KeyData** - Information about the document's key.
    
    - **_Arg** - Static argument passed by the caller into the `map` phase.

- **lines 20-21** - Turn the Riak document into a structure. We store our data as json so we use `mochijson:decode` to transform the data.

- **lines 23-28** - Define a function to help with picking the data that matches the supplied argument(s).

- **lines 30-34** - Use the supplied argument to filter the data we want. Return an empty array if the data doesn't match our query.

- **line 40** - Again with the function definitions and arguments.
    - **Values** - Result from the map phase.
    
    - **_Arg** - Static argument passed by the caller into the `reduce` phase.

- **line 41** - Artificially limit the results by a supplied argument.

### General Processing Notes

- **line 49** - Using a list of keys will help performance greatly. See below for suggestions on obtaining a list of keys.

- **line 52** - To help with VM memory, and to imitate so semblance of transactionality, I run things in batches.

- **lines 54-56** - This defines the job iteslf. The `map` phase or the `reduce` phase can be ommitted but not both. The `keep` argument determines whether the documents from this phase are included in the final output. I tend to use `keep=true` only on the last phase run (i.e. `map` phase if no `reduce`; `reduce` phase otherwise)

- **line 58** - Add each `bucket`/`key` pair to the job. The key can be excluded specifying just the bucket but this will cause a full bucket scan. I've had problems with blowing out the VM memory when not specifying the keys and iterating a large bucket.

### Obtaining Document Keys

I get a list of keys independent of my map reduce job to take some of the overhead out of the MR job itself. I've used a couple of different methods to get a list of keys to process.

1. ****Query keys from an index**.** If you have an index you can lean on, it will cut down lots of overhead to query that index for keys to work through.

3. [**Stream key list from the bucket**](http://docs.basho.com/riak/1.2.1/references/apis/http/HTTP-List-Keys/#Request). This can take a while and add significant overhead to the ring, but is useful if you need all keys in the bucket. When streaming keys, the multipart response produces what looks like multiple documents. The pattern, `{"keys": [..]}`, will show up multiple times and require some massaging before it can be read by a JSON reader.

Given my ignorance of Erlang and Riak, I'm sure there are better ways to accomplish these same steps. I've performed the same work using JavaScript with like results, but JavaScript is handled outside of the Erlang VM and doesn't tend to be as fast as Erlang. For M/R jobs that you will be running more than once, it is suggested to compile the Erlang functions and putting the libraries on the servers in the ring.

At this point, I'm just parroting what I've heard and can only cause confusion saying more. Hopefully this is more helpful than confusing. Be sure to check #riak on Freenode for some really great people and help.
