---
layout: post
title: Rsyslog to Elasticsearch 
categories:
header_image: /img/elastic.jpg
header_permalink: https://flic.kr/p/dtMQd 
---

# {{ page.title }}

For decades systems administrations have known that it's important to centralize logs, be it for troubleshooting or security reasons. In my case, not only do I want to centralize logs, but I also want them to be searchable. (Not that grep on a centralized log file isn't powerful, but that's not the solution I'm looking for at this time.) 

## Elasticsearch

Elasticsearch has also been around for a while. 

Elasticsearch:

> ... is a distributed, RESTful search and analytics engine capable of solving a growing number of use cases. As the heart of the Elastic Stack, it centrally stores your data so you can discover the expected and uncover the unexpected.

Bascially it allows storing, indexing, and searching data. Typically it's part of a stack, the "ELK" stack, made up of Elasticsearch, Logstash, and Kibana. 

As a systems administrator, I think it's also important to note that Elasticsearch is, when used as a cluster, a distributed system, which means Aphyr has [tested it](https://aphyr.com/posts/323-jepsen-elasticsearch-1-5-0). One shouldn't take the addition of yet another distributed system lightly. As an example, I have MySQL Galera, RabbitMQ, Nomad, and now Elasticsearch for distributed systems in production right now.

## Rsyslog directly to Elasticsearch

The point of this post is to show how to use rsyslog to send logs directly into an Elasticsearch cluster. Currently I am not using the L part of the stack, meaning I have no Logstash. I'm just using rsyslog to send log messages directly into Elasticsearch, and I use Kibana as a graphical interface to search logs. I would imagine that in the future I will use Logstash, but for now I am not.

I've put up a very simple [Ansible role](https://github.com/ccollicutt/ansible-role-elasticsearch-rsyslog) to configure rsyslog to send logs to Elasticsearch. I run Haproxy in front of the Elasticsearch cluster so I send all the logs to the Haproxy virtual IP, which loadbalances across the Elasticsearch cluster. But other than that, this is what I use, and it has been taking in millions of logs, probably 30/second, for a few weeks.

## omelasticsearch

omelasticsearch does all the work. You just need a rsyslog version recent enough to have that module. Then add in a configuration file which points to your Elasticsearch cluster, and you're done. Quite simple actually.

Here is the package:

~~~bash
$ dpkg --list rsyslog-elasticsearch
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version                 Architecture            Description
+++-=====================================-=======================-=======================-===============================================================================
ii  rsyslog-elasticsearch                 8.16.0-1ubuntu3         amd64                   Elasticsearch output plugin for rsyslog
~~~

And here is the configuration file:

~~~bash
{% raw %}$ cat templates/etc/rsyslog.d/elasticsearch.conf 
module(load="omelasticsearch") # Elasticsearch output module

# this is for index names to be like: logstash-YYYY.MM.DD
template(name="logstash-index"
  type="list") {
    constant(value="logstash-")
    property(name="timereported" dateFormat="rfc3339" position.from="1" position.to="4")
    constant(value=".")
    property(name="timereported" dateFormat="rfc3339" position.from="6" position.to="7")
    constant(value=".")
    property(name="timereported" dateFormat="rfc3339" position.from="9" position.to="10")
}


# template to generate JSON documents for Elasticsearch in Logstash format
template(name="plain-syslog"
  type="list") {
    constant(value="{")
    constant(value="\"@timestamp\":\"")         property(name="timereported" dateFormat="rfc3339")
    constant(value="\",\"host\":\"")            property(name="hostname")
    constant(value="\",\"severity-num\":")      property(name="syslogseverity")
    constant(value=",\"facility-num\":")        property(name="syslogfacility")
    constant(value=",\"severity\":\"")          property(name="syslogseverity-text")
    constant(value="\",\"facility\":\"")        property(name="syslogfacility-text")
    constant(value="\",\"syslogtag\":\"")       property(name="syslogtag" format="json")
    constant(value="\",\"message\":\"")         property(name="msg" format="json")
    constant(value="\"}")
  }

action(type="omelasticsearch"
  server="{{ elastic_search_ip }}"
  serverport="9200"
  template="plain-syslog"  # use the template defined earlier
  searchIndex="logstash-index"
  dynSearchIndex="on"
  searchType="events"
  bulkmode="on"                   # use the Bulk API
  queue.dequeuebatchsize="5000"   # ES bulk size
  queue.size="100000"   # capacity of the action queue
  queue.workerthreads="5"   # 5 workers for the action
  action.resumeretrycount="-1"  # retry indefinitely if ES is unreachable
  errorfile="/var/log/omelasticsearch.log"
) {% endraw %}
~~~

<br>

## Curator

You will probably want to prune the logs that are entered into your ES cluster, that is unless you have a lot of storage space. I am using [Curator](https://github.com/elastic/curator) to do that. The omelastic module configuration show above adds indices with names like "logstash-YYY-MM-DD" and you can use that pattern with a curator action file to delete indices older than a certain number of days. I have a daily job that is run out of a Nomad cluster to prune indices over 30 days old.


~~~bash
$ curl 'elasticsearch-vip:9200/_cat/indices?v'
health status index               pri rep docs.count docs.deleted store.size pri.store.size 
SNIP!
green  open   logstash-2016.11.04   5   1     158184            0     55.4mb         27.7mb 
green  open   .kibana               1   1          7            1     72.1kb           36kb 
green  open   logstash-2016.11.05   5   1     538415            0      181mb         90.5mb 
green  open   logstash-2016.11.06   5   1     538909            0    181.1mb         90.5mb 
green  open   logstash-2016.11.07   5   1     591699            0    219.6mb        109.9mb 
green  open   logstash-2016.11.08   5   1     574186            0      207mb        103.5mb 
green  open   logstash-2016.11.09   5   1     626842            0    231.8mb        115.9mb 
green  open   logstash-2016.11.10   5   1     570544            0    203.8mb        101.9mb 
green  open   logstash-2016.11.11   5   1     350968            0      145mb         69.9mb 
~~~ 

<br>

## Performance

I should also note that this configuration may not be as performant as you would like. There are certainly considerations to make in terms of the number of indices, and shards, and their size, and how that relates to performance of the ES cluster over time. I would doubt that this example will be useful in large systems and would require a fair amount of tuning. Please keep that in mind. :)

## Conclusion

This is a pretty simplistic example of Elasticsearch use. But I think it's helpful because it can get you up and running, logging to Elasticsearch, quickly, and from this point you can work on improving things as you learn about ES. That's my plan anyways. :)
