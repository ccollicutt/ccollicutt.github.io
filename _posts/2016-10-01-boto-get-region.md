---
layout: post
title: Getting an AWS Instance's Region with Boto
categories:
header_image: /img/dolphin.jpg
header_permalink: https://flic.kr/p/5kSYhr 
---

# {{ page.title }}

This is a quick post on how to get the region an AWS instance is in using Boto.

## tl;dr

I'm using a virtualenv with boto installed in an instance in AWS. To use *boto.ec2.connect_to_region()* I need to provide the region. But how do I know that without having to hard code it or get it from a config file? I can use the instances AWS metadata.

~~~bash
(venv) ubuntu@ip-172-31-14-171:~$ cat region.py 
#!/opt/venv/bin/python2.7

import boto.utils

data = boto.utils.get_instance_identity()
region_name = data['document']['region']

print region_name
(venv) ubuntu@ip-172-31-14-171:~$ ./region.py 
us-west-1
~~~

Now I know what region the instance is in programmatically.

## A bit more

AWS instances can have an IAM role that allows them to do many things in AWS, such as creating instances or uploading files to S3 or creating "virtual private clouds" (VPCs, ie. networking). They don't have to have API IDs or keys, they can just be given roles on creation. Obviously, once they are deleted they no longer have that access.

Further, the Boto library is smart enough to know when it is running in an AWS instance.

This way we can list all ec2 instances from within a particular instance without having to provide it specific credentials. However, in order to connect to ec2 using Boto we need to provide the region. As mentioned, thankfully Boto provides a way to do that prior to connecting using the instances metadata.

~~~bash
(venv) ubuntu@ip-172-31-14-171:~$ cat connect.py 
#!/opt/venv/bin/python2.7

import boto.utils
import boto.ec2

data = boto.utils.get_instance_identity()
region_name = data['document']['region']

conn = boto.ec2.connect_to_region(region_name)

for i in conn.get_only_instances():
   print i
(venv) ubuntu@ip-172-31-14-171:~$ ./connect.py 
Instance:i-5ed52dea
~~~

That's it! Now depending on the roles and permissions the instance has in terms of AWS, it can use those APIs to manage all kinds of instructure and tools.

Hat tip to this [Stackoverflow](http://stackoverflow.com/questions/21365408/boto-python-library-still-thinks-it-is-in-original-ec2-datacentre-after-migratio_) post.
