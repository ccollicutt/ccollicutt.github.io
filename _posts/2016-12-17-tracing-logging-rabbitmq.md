---
layout: post
title: Tracing or Logging RabbitMQ
categories:
header_image: /img/rabbitmq.jpg
---

# {{ page.title }}

Messaging queues like [RabbitMQ](https://www.rabbitmq.com/) are widely used. For example it is a key component in an OpenStack deployment.

Sometimes you need to peek into the queue and see what is happening and I thought I would write a quick post on one way to do that.

## Turn on the firehose

Rabbit has an option to turn on "tracing" so that you can see every message that is coming through the queue.

>RabbitMQ has a "firehose" feature, where the administrator can enable (on a per-node, per-vhost basis) an exchange to which publish- and delivery-notifications should be CCed. - [RabbitMQ](http://www.rabbitmq.com/firehose.html)

If you have a default install and just want the "/" vhost, you can simply turn on tracing.

~~~bash
root@rabbit:~# rabbitmqctl trace_on 
Starting tracing for vhost "/" ...
~~~

Now you need to read that queue.

## Reading messages on the firehose queue

First you will need a script to read the queue.

I just grabbed one off of github, but it would be straight forward to write your own.

<script src="https://gist.github.com/khomenko/2562165.js"></script>

First, this particular script uses the pika library.

~~~bash
ubuntu@rabbit:~$ virtualenv venv
ubuntu@rabbit:~$ . venv/bin/activate
ubuntu@rabbit:~$ pip install pika
~~~

Now you can turn on the firehost. I'm going to redirect to a file because there are a lot of messages.

~~~bash
(venv) ubuntu@rabbit:~$ time ./trace.py > trace.out
^CTraceback (most recent call last):
  File "./trace.py", line 37, in <module>
    channel.start_consuming()
  File "/home/ubuntu/venv/local/lib/python2.7/site-packages/pika/adapters/blocking_connection.py", line 1681, in start_consuming
    self.connection.process_data_events(time_limit=None)
  File "/home/ubuntu/venv/local/lib/python2.7/site-packages/pika/adapters/blocking_connection.py", line 647, in process_data_events
    self._flush_output(common_terminator)
  File "/home/ubuntu/venv/local/lib/python2.7/site-packages/pika/adapters/blocking_connection.py", line 410, in _flush_output
    self._impl.ioloop.poll()
  File "/home/ubuntu/venv/local/lib/python2.7/site-packages/pika/adapters/select_connection.py", line 590, in poll
    events = self._poll.poll(self.get_next_deadline())
KeyboardInterrupt

real    0m2.421s
user    0m0.096s
sys 0m0.024s
(venv) ubuntu@rabbit:~$ wc -l trace.out 
99 trace.out
(venv) ubuntu@rabbit:~$ head trace.out 
 [*] Waiting for logs. To exit press CTRL+C
 [x] 'publish.nova':u'rabbit@rabbit':[u'conductor']:'{"oslo.message": "{\\"_context_domain\\": null, \\"_msg_id\\": \\"91deacb97e0948069f234db946d661ae\\", \\"_context_quota_class\\": null, \\"_context_read_only\\": false, \\"_context_request_id\\": \\"req-e282aa6c-ccd0-4a23-aa1c-e818a339c56a\\", \\"_context_service_catalog\\": [], \\"args\\": {\\"objmethod\\": \\"save\\", \\"args\\": [], \\"objinst\\": {\\"nova_object.version\\": \\"1.20\\", \\"nova_object.changes\\": [\\"report_count\\"], \\"nova_object.name\\": \\"Service\\", \\"nova_object.data\\": {\\"binary\\": \\"nova-compute\\", \\"deleted\\": false, \\"created_at\\": \\"2016-12-13T21:53:45Z\\", \\"updated_at\\": \\"2016-12-17T15:31:28Z\\", \\"report_count\\": 32164, \\"topic\\": \\"compute\\", \\"host\\": \\"server03\\", \\"version\\": 15, \\"disabled\\": false, \\"forced_down\\": false, \\"last_seen_up\\": \\"2016-12-17T15:31:28Z\\", \\"deleted_at\\": null, \\"disabled_reason\\": null, \\"id\\": 7}, \\"nova_object.namespace\\": \\"nova\\"}, \\"kwargs\\": {}}, \\"_unique_id\\": \\"3d3aaa47d8b94fc8bf025a797e788d87\\", \\"_context_resource_uuid\\": null, \\"_context_instance_lock_checked\\": false, \\"_context_is_admin_project\\": true, \\"_context_user\\": null, \\"_context_user_id\\": null, \\"_context_project_name\\": null, \\"_context_read_deleted\\": \\"no\\", \\"_context_user_identity\\": \\"- - - - -\\", \\"_reply_q\\": \\"reply_0949e6a10e0345c4ba494eb121edc1f1\\", \\"_context_auth_token\\": null, \\"_context_show_deleted\\": false, \\"_context_tenant\\": null, \\"_context_roles\\": [], \\"_context_is_admin\\": true, \\"version\\": \\"3.0\\", \\"_context_project_id\\": null, \\"_context_project_domain\\": null, \\"_context_timestamp\\": \\"2016-12-16T16:05:25.637936\\", \\"_context_user_domain\\": null, \\"_context_user_name\\": null, \\"method\\": \\"object_action\\", \\"_context_remote_address\\": null}", "oslo.version": "2.0"}'
 [x] 'deliver.conductor':u'rabbit@rabbit':[u'conductor']:'{"oslo.message": "{\\"_context_domain\\": null, \\"_msg_id\\": \\"91deacb97e0948069f234db946d661ae\\", \\"_context_quota_class\\": null, \\"_context_read_only\\": false, \\"_context_request_id\\": \\"req-e282aa6c-ccd0-4a23-aa1c-e818a339c56a\\", \\"_context_service_catalog\\": [], \\"args\\": {\\"objmethod\\": \\"save\\", \\"args\\": [], \\"objinst\\": {\\"nova_object.version\\": \\"1.20\\", \\"nova_object.changes\\": [\\"report_count\\"], \\"nova_object.name\\": \\"Service\\", \\"nova_object.data\\": {\\"binary\\": \\"nova-compute\\", \\"deleted\\": false, \\"created_at\\": \\"2016-12-13T21:53:45Z\\", \\"updated_at\\": \\"2016-12-17T15:31:28Z\\", \\"report_count\\": 32164, \\"topic\\": \\"compute\\", \\"host\\": \\"server03\\", \\"version\\": 15, \\"disabled\\": false, \\"forced_down\\": false, \\"last_seen_up\\": \\"2016-12-17T15:31:28Z\\", \\"deleted_at\\": null, \\"disabled_reason\\": null, \\"id\\": 7}, \\"nova_object.namespace\\": \\"nova\\"}, \\"kwargs\\": {}}, \\"_unique_id\\": \\"3d3aaa47d8b94fc8bf025a797e788d87\\", \\"_context_resource_uuid\\": null, \\"_context_instance_lock_checked\\": false, \\"_context_is_admin_project\\": true, \\"_context_user\\": null, \\"_context_user_id\\": null, \\"_context_project_name\\": null, \\"_context_read_deleted\\": \\"no\\", \\"_context_user_identity\\": \\"- - - - -\\", \\"_reply_q\\": \\"reply_0949e6a10e0345c4ba494eb121edc1f1\\", \\"_context_auth_token\\": null, \\"_context_show_deleted\\": false, \\"_context_tenant\\": null, \\"_context_roles\\": [], \\"_context_is_admin\\": true, \\"version\\": \\"3.0\\", \\"_context_project_id\\": null, \\"_context_project_domain\\": null, \\"_context_timestamp\\": \\"2016-12-16T16:05:25.637936\\", \\"_context_user_domain\\": null, \\"_context_user_name\\": null, \\"method\\": \\"object_action\\", \\"_context_remote_address\\": null}", "oslo.version": "2.0"}'
 [x] 'publish.reply_0949e6a10e0345c4ba494eb121edc1f1':u'rabbit@rabbit':[u'reply_0949e6a10e0345c4ba494eb121edc1f1']:'{"oslo.message": "{\\"_msg_id\\": \\"91deacb97e0948069f234db946d661ae\\", \\"failure\\": null, \\"_unique_id\\": \\"fbb674f006f54c939dd3df7b8e234748\\", \\"result\\": [{\\"last_seen_up\\": \\"2016-12-17T15:31:38Z\\", \\"updated_at\\": \\"2016-12-17T15:31:38Z\\", \\"obj_what_changed\\": []}, null], \\"ending\\": true}", "oslo.version": "2.0"}'
 [x] 'deliver.reply_0949e6a10e0345c4ba494eb121edc1f1':u'rabbit@rabbit':[u'reply_0949e6a10e0345c4ba494eb121edc1f1']:'{"oslo.message": "{\\"_msg_id\\": \\"91deacb97e0948069f234db946d661ae\\", \\"failure\\": null, \\"_unique_id\\": \\"fbb674f006f54c939dd3df7b8e234748\\", \\"result\\": [{\\"last_seen_up\\": \\"2016-12-17T15:31:38Z\\", \\"updated_at\\": \\"2016-12-17T15:31:38Z\\", \\"obj_what_changed\\": []}, null], \\"ending\\": true}", "oslo.version": "2.0"}'
 [x] 'publish.nova':u'rabbit@rabbit':[u'conductor']:'{"oslo.message": "{\\"_context_domain\\": null, \\"_msg_id\\": \\"1db1384294174e8d83ba30ffae7912e2\\", \\"_context_quota_class\\": null, \\"_context_read_only\\": false, \\"_context_request_id\\": \\"req-e8d0d0c9-35b1-4d47-9ab9-b0d6f08283eb\\", \\"_context_service_catalog\\": [], \\"args\\": {\\"objmethod\\": \\"save\\", \\"args\\": [], \\"objinst\\": {\\"nova_object.version\\": \\"1.20\\", \\"nova_object.changes\\": [\\"report_count\\"], \\"nova_object.name\\": \\"Service\\", \\"nova_object.data\\": {\\"binary\\": \\"nova-compute\\", \\"deleted\\": false, \\"created_at\\": \\"2016-12-13T21:50:18Z\\", \\"updated_at\\": \\"2016-12-17T15:31:29Z\\", \\"report_count\\": 32176, \\"topic\\": \\"compute\\", \\"host\\": \\"server02\\", \\"version\\": 15, \\"disabled\\": false, \\"forced_down\\": false, \\"last_seen_up\\": \\"2016-12-17T15:31:29Z\\", \\"deleted_at\\": null, \\"disabled_reason\\": null, \\"id\\": 6}, \\"nova_object.namespace\\": \\"nova\\"}, \\"kwargs\\": {}}, \\"_unique_id\\": \\"d484f67b6dc34bc88b26c7a011015c36\\", \\"_context_resource_uuid\\": null, \\"_context_instance_lock_checked\\": false, \\"_context_is_admin_project\\": true, \\"_context_user\\": null, \\"_context_user_id\\": null, \\"_context_project_name\\": null, \\"_context_read_deleted\\": \\"no\\", \\"_context_user_identity\\": \\"- - - - -\\", \\"_reply_q\\": \\"reply_eab5dc5ef70a480881932f27d8d2157b\\", \\"_context_auth_token\\": null, \\"_context_show_deleted\\": false, \\"_context_tenant\\": null, \\"_context_roles\\": [], \\"_context_is_admin\\": true, \\"version\\": \\"3.0\\", \\"_context_project_id\\": null, \\"_context_project_domain\\": null, \\"_context_timestamp\\": \\"2016-12-16T16:05:22.325724\\", \\"_context_user_domain\\": null, \\"_context_user_name\\": null, \\"method\\": \\"object_action\\", \\"_context_remote_address\\": null}", "oslo.version": "2.0"}'
 [x] 'deliver.conductor':u'rabbit@rabbit':[u'conductor']:'{"oslo.message": "{\\"_context_domain\\": null, \\"_msg_id\\": \\"1db1384294174e8d83ba30ffae7912e2\\", \\"_context_quota_class\\": null, \\"_context_read_only\\": false, \\"_context_request_id\\": \\"req-e8d0d0c9-35b1-4d47-9ab9-b0d6f08283eb\\", \\"_context_service_catalog\\": [], \\"args\\": {\\"objmethod\\": \\"save\\", \\"args\\": [], \\"objinst\\": {\\"nova_object.version\\": \\"1.20\\", \\"nova_object.changes\\": [\\"report_count\\"], \\"nova_object.name\\": \\"Service\\", \\"nova_object.data\\": {\\"binary\\": \\"nova-compute\\", \\"deleted\\": false, \\"created_at\\": \\"2016-12-13T21:50:18Z\\", \\"updated_at\\": \\"2016-12-17T15:31:29Z\\", \\"report_count\\": 32176, \\"topic\\": \\"compute\\", \\"host\\": \\"server02\\", \\"version\\": 15, \\"disabled\\": false, \\"forced_down\\": false, \\"last_seen_up\\": \\"2016-12-17T15:31:29Z\\", \\"deleted_at\\": null, \\"disabled_reason\\": null, \\"id\\": 6}, \\"nova_object.namespace\\": \\"nova\\"}, \\"kwargs\\": {}}, \\"_unique_id\\": \\"d484f67b6dc34bc88b26c7a011015c36\\", \\"_context_resource_uuid\\": null, \\"_context_instance_lock_checked\\": false, \\"_context_is_admin_project\\": true, \\"_context_user\\": null, \\"_context_user_id\\": null, \\"_context_project_name\\": null, \\"_context_read_deleted\\": \\"no\\", \\"_context_user_identity\\": \\"- - - - -\\", \\"_reply_q\\": \\"reply_eab5dc5ef70a480881932f27d8d2157b\\", \\"_context_auth_token\\": null, \\"_context_show_deleted\\": false, \\"_context_tenant\\": null, \\"_context_roles\\": [], \\"_context_is_admin\\": true, \\"version\\": \\"3.0\\", \\"_context_project_id\\": null, \\"_context_project_domain\\": null, \\"_context_timestamp\\": \\"2016-12-16T16:05:22.325724\\", \\"_context_user_domain\\": null, \\"_context_user_name\\": null, \\"method\\": \\"object_action\\", \\"_context_remote_address\\": null}", "oslo.version": "2.0"}'
 [x] 'publish.reply_eab5dc5ef70a480881932f27d8d2157b':u'rabbit@rabbit':[u'reply_eab5dc5ef70a480881932f27d8d2157b']:'{"oslo.message": "{\\"_msg_id\\": \\"1db1384294174e8d83ba30ffae7912e2\\", \\"failure\\": null, \\"_unique_id\\": \\"366886d2234e49649665576f43ae3f72\\", \\"result\\": [{\\"last_seen_up\\": \\"2016-12-17T15:31:39Z\\", \\"updated_at\\": \\"2016-12-17T15:31:39Z\\", \\"obj_what_changed\\": []}, null], \\"ending\\": true}", "oslo.version": "2.0"}'
 [x] 'deliver.reply_eab5dc5ef70a480881932f27d8d2157b':u'rabbit@rabbit':[u'reply_eab5dc5ef70a480881932f27d8d2157b']:'{"oslo.message": "{\\"_msg_id\\": \\"1db1384294174e8d83ba30ffae7912e2\\", \\"failure\\": null, \\"_unique_id\\": \\"366886d2234e49649665576f43ae3f72\\", \\"result\\": [{\\"last_seen_up\\": \\"2016-12-17T15:31:39Z\\", \\"updated_at\\": \\"2016-12-17T15:31:39Z\\", \\"obj_what_changed\\": []}, null], \\"ending\\": true}", "oslo.version": "2.0"}'
 [x] 'publish.nova':u'rabbit@rabbit':[u'conductor']:'{"oslo.message": "{\\"_context_domain\\": null, \\"_msg_id\\": \\"06d003ee9aa0445e8947c27a92275f0c\\", \\"_context_quota_class\\": null, \\"_context_read_only\\": false, \\"_context_request_id\\": \\"req-dc8f9ccf-151e-4f9b-9fd9-52350c39fd6f\\", \\"_context_service_catalog\\": [], \\"args\\": {\\"object_versions\\": {\\"PciDevicePoolList\\": \\"1.1\\", \\"ServiceList\\": \\"1.19\\", \\"PciDevicePool\\": \\"1.1\\", \\"Service\\": \\"1.20\\", \\"TagList\\": \\"1.1\\", \\"InstancePCIRequests\\": \\"1.1\\", \\"VirtCPUModel\\": \\"1.0\\", \\"MigrationContext\\": \\"1.1\\", \\"SecurityGroup\\": \\"1.1\\", \\"DeviceBus\\": \\"1.0\\", \\"Instance\\": \\"2.3\\", \\"KeyPair\\": \\"1.4\\", \\"KeyPairList\\": \\"1.3\\", \\"DeviceMetadata\\": \\"1.0\\", \\"InstancePCIRequest\\": \\"1.1\\", \\"EC2Ids\\": \\"1.0\\", \\"InstanceInfoCache\\": \\"1.5\\", \\"VirtCPUFeature\\": \\"1.0\\", \\"Flavor\\": \\"1.1\\", \\"SecurityGroupList\\": \\"1.0\\", \\"PciDevice\\": \\"1.5\\", \\"VirtCPUTopology\\": \\"1.0\\", \\"InstanceNUMACell\\": \\"1.3\\", \\"InstanceList\\": \\"2.1\\", \\"ComputeNode\\": \\"1.16\\", \\"InstanceFault\\": \\"1.2\\", \\"Tag\\": \\"1.1\\", \\"HVSpec\\": \\"1.2\\", \\"InstanceDeviceMetadata\\": \\"1.0\\", \\"InstanceNUMATopology\\": \\"1.2\\", \\"PciDeviceList\\": \\"1.3\\"}, \\"objmethod\\": \\"get_by_host\\", \\"args\\": [\\"server03\\"], \\"objname\\": \\"InstanceList\\", \\"kwargs\\": {\\"use_slave\\": true, \\"expected_attrs\\": []}}, \\"_unique_id\\": \\"42f089b774994641a15621df44b4493e\\", \\"_context_resource_uuid\\": null, \\"_context_instance_lock_checked\\": false, \\"_context_is_admin_project\\": true, \\"_context_user\\": null, \\"_context_user_id\\": null, \\"_context_project_name\\": null, \\"_context_read_deleted\\": \\"no\\", \\"_context_user_identity\\": \\"- - - - -\\", \\"_reply_q\\": \\"reply_0949e6a10e0345c4ba494eb121edc1f1\\", \\"_context_auth_token\\": null, \\"_context_show_deleted\\": false, \\"_context_tenant\\": null, \\"_context_roles\\": [\\"admin\\"], \\"_context_is_admin\\": true, \\"version\\": \\"3.0\\", \\"_context_project_id\\": null, \\"_context_project_domain\\": null, \\"_context_timestamp\\": \\"2016-12-17T15:31:43.661237\\", \\"_context_user_domain\\": null, \\"_context_user_name\\": null, \\"method\\": \\"object_class_action_versions\\", \\"_context_remote_address\\": null}", "oslo.version": "2.0"}'
~~~

Once you've got the messages you want, turn off tracing.

~~~bash
root@rabbit:~# rabbitmqctl trace_off
Stopping tracing for vhost "/" ...
~~~

You might need to empty out that queue once you are done.

~~~bash
root@rabbit:~# rabbitmqctl list_queues | grep firehose
firehose    450
root@rabbit:~# rabbitmqctl purge_queue firehose
Purging queue 'firehose' in vhost '/' ...
root@rabbit:~# rabbitmqctl list_queues | grep firehose
firehose    0
~~~
