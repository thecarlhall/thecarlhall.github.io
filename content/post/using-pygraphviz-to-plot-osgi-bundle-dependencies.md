---
title: "Using pygraphviz to plot OSGi bundle dependencies"
date: "2011-09-05"
categories: 
  - "development"
  - "osgi"
---

I've been working on an OSGi project for the last few years. As with any project, evolutionary changes will eventually require some cleanup. As new bundles have been added over time, the graph of dependencies is starting to get unwieldy in places. Even with good management of these dependencies, a nice visual layout of things can really help you see how your bundles are interconnected and give you the power to start separating some connections if you graph starts to hit cyclical dependencies.

<!--more-->

I drove around a few different visual tools in Eclipse ([PDE visualization tool](http://www.eclipse.org/pde/incubator/dependency-visualization/ "PDE visualization tool"), m2eclipse dependency graph) but needed something that would narrow the view to just my project. I not only wanted to see what things my bundles depend on in the project but I wanted to see what depends on a given bundle.

This was my first project with pygraphviz but after I figured out which way the grain runs, I was able to give it a basic graph of my project and generate the diagrams I wanted. I finally let pygraphviz handle the graph traversing and things got better (less code, faster results).

Since I wanted to analyze the runtime dependencies of my server, I installed the [Felix Remote Shell](http://felix.apache.org/site/apache-felix-remote-shell.html "Felix Remote Shell") bundle along with the [Felix Shell](http://felix.apache.org/site/apache-felix-shell.html "Felix Shell") bundle to allow remote connections to management functionality. With these in place, I was able to connect via telnet and query for package level information to build my graph.

Using the code below, I was able to generate a graph of the entire project (successors) and a graph for each bundle in the graph (predecessors and successors). Some sample images are below the source code. Eventually I'll add saving and opening of dot files to allow for analysis without a running server.

```python
#! /usr/bin/env python
import re
from sets import Set
import os
import pygraphviz as pgv
import sys
import telnetlib

# [   1] [Active     ] [   15] org.sakaiproject.nakamura.messaging (0.11.0.SNAPSHOT)
bundle_from_ps = re.compile('^\[\s*(?P&lt;bundle_id&gt;\d+)\]\s\[.+\]\s(?P&lt;bundle_name&gt;.+)\s')

# org.sakaiproject.nakamura.api.solr; version=0.0.0 -&gt; org.sakaiproject.nakamura.solr [11]
bundle_from_req = re.compile('^.*-\&gt; (?P&lt;bundle_name&gt;.*) \[(?P&lt;bundle_id&gt;.*)\]$')

def get_sakai_bundles():
    &quot;&quot;&quot;Get a list of bundles that are create as part of Sakai OAE.
    Returns a dictionary of dict[bundle_name] = bundle_id.
    &quot;&quot;&quot;
    tn = telnetlib.Telnet('localhost', '6666')
    tn.write('ps -s\nexit\n')
    lines = [line for line in tn.read_all().split('\r\n') if 'org.sakaiproject' in line]

    bundles = {}
    for line in lines:
        m = bundle_from_ps.match(line)
        bundles[m.group('bundle_name')] = m.group('bundle_id')
    return bundles

def get_package_reqs(bundle_id):
    &quot;&quot;&quot;Gets the requirements (imports) for a given bundle.
    Returns a dictionary of dict[bundle_name] = bundle_id.

    Keyword arguments:
    bundle_id -- Bundle ID returned by the server in the output of
                 get_sakai_bundles()
    &quot;&quot;&quot;
    tn = telnetlib.Telnet('localhost', '6666')
    tn.write('inspect package requirement %s\nexit\n' % (bundle_id))
    lines = [line for line in tn.read_all().split('\r\n') if line.startswith('org.sakaiproject') and line.endswith(']')]

    reqs = {}
    for line in lines:
        m = bundle_from_req.match(line)
        reqs[m.group('bundle_name')] = m.group('bundle_id')
    return reqs

def build_bundle_graph():
    &quot;&quot;&quot;Build a graph_attr (nodes, edges) representing the connectivity
    within Sakai bundles
    &quot;&quot;&quot;
    sakai_bundles = get_sakai_bundles()
    bundles = {}
    for b_name, b_id in sakai_bundles.items():
        reqs = get_package_reqs(b_id)
        bundles[b_name] = reqs.keys()
    return bundles

def draw_subgraph(name, graph, filename, successors = True):
    &quot;&quot;&quot;Draw (write to disk) a subgraph that starts with or ends with
    the specified node.

    Keyword arguments:
    name -- name of the node to focus on
    graph -- graph of paths between bundles
    filename -- filename to write subgraph to
    successors -- whether to lookup successors or predecessors
    &quot;&quot;&quot;
    if successors:
        nbunch = graph.successors(name)
    else:
        nbunch = graph.predecessors(name)

    if nbunch:
        nbunch.append(name)
        subgraph = graph.subgraph(nbunch)
        subgraph.layout(prog = 'dot')
        subgraph.draw('graphviz/%s' % filename)

def main():
    if not os.path.isdir('graphviz'):
        os.mkdir('graphviz')

    bundles_graph = build_bundle_graph()

    # print the whole graph
    graph = pgv.AGraph(data = bundles_graph, directed = True)
    graph.layout(prog = 'dot')
    graph.draw('graphviz/org.sakaiproject.nakamura.png')

    for b_name, reqs in bundles_graph.items():
        draw_subgraph(b_name, graph, '%s.png' % b_name)
        draw_subgraph(b_name, graph, '%s-pred.png' % b_name, False)

if __name__ == '__main__':
    main()
```

<figure>

![Sakai OAE Bundle Dependency Graph](/images/org-sakaiproject-nakamura.png)

<figcaption>

Sakai OAE Bundle Dependency Graph

</figcaption>

</figure>

<figure>

![Sakai OAE Solr Bundle Predecessors](/images/org-sakaiproject-nakamura-solr-pred.png)

<figcaption>

Sakai OAE Solr Bundle Predecessors

</figcaption>

</figure>

<figure>

![Sakai OAE Presence Bundle Successors](/images/org-sakaiproject-nakamura-presence.png)

<figcaption>

Sakai OAE Presence Bundle Successors

</figcaption>

</figure>
