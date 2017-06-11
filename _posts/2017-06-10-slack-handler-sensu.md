---
layout: post
title: Setting up a Sensu Slack Handler
categories:
header_image: /img/nyc-glitch.jpg
header_permalink: https://flic.kr/p/qszML1
---

# {{ page.title }}

I've been working on a Sensu install recently. I had some trouble getting the Slack handler working. The docs were a little off in terms of how to do this. But I can now say that it's working. :)

First, you will need the sensu-plugins-slack plugin.

I have Sensu embedded, so I use that gem to install it. It's already been installed below; I'm just showing the command.

```
# /opt/sensu/embedded/bin/gem install sensu-plugins-slack
You can use the embedded Ruby by setting EMBEDDED_RUBY=true in /etc/default/sensu
Successfully installed sensu-plugins-slack-1.0.0
Parsing documentation for sensu-plugins-slack-1.0.0
Installing ri documentation for sensu-plugins-slack-1.0.0
Done installing documentation for sensu-plugins-slack after 0 seconds
1 gem installed
```

Next, login to your slack account and create a webhook.

Now to configure it to be used.

```
# cat /etc/sensu/conf.d/slack_handler.json
{
  "handlers": {
    "slack": {
      "type": "pipe",
      "command": "handler-slack.rb",
      "severites": ["critical", "unknown"]
      }
    },
    "slack": {
      "webhook_url": "<your slack webhook url>",
      "template" : "",
      "channel": "#general"
    }
}
```

I have also setup Slack to be a default handler.

```
# cat /etc/sensu/conf.d/default_handlers.json
{
  "handlers": {
    "default": {
      "type": "set",
      "handlers": [
        "slack"
      ]
    }
  }
}
```

Once that is setup and Sensu restarted, you should be able to get alerts into a Slack channel.

Happy Sensu Slacking!
