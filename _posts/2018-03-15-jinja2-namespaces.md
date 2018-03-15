---
layout: post
title: Jinja2 Namespaces and Variable Scope
categories:
---

# {{ page.title }}

I have been doing some Network automation lately, specifically using the Juniper vSRX and generating config templates via Ansible.

I quickly found out about variable scope in Jinja.

>Please keep in mind that it is not possible to set variables inside a block and have them show up outside of it. This also applies to loops. The only exception to that rule are if statements which do not introduce a scope. - [Jinja Docs](http://jinja.pocoo.org/docs/2.10/templates/)

Putting a lot of logic into templates is probably not a good idea, but I'm doing it anyway in this case. :)

Example. I have two friends. One has a car, one doesn't. Does the group of friends have a car?

```
{% raw %}
#!/usr/bin/env python

from jinja2 import Template

friends = []

friend1 = {
    "name": "Frank",
    "car": False
}
friend2 = {
    "name": "Anna",
    "car": True
}

friends.append(friend1)
friends.append(friend2)

t_example = """
{% set ns = namespace(hasCar=false) %}
{% for f in friends %}
Friend {{ f.name }}
{% if f.car == true %}
{% set ns.hasCar = true %}
{% endif %}
{% endfor %} {# for n in friends #}

{% if ns.hasCar == true %}
SOMEONE HAS A CAR: TRUE
{% else %}
SOMEONE HAS A CAR: FALSE
{% endif %}
{% endraw %}
"""

template = Template(t_example, trim_blocks=True)

render = template.render(friends=friends)

print render
```

If I run this in ipython I get:

```

In [1]: run jinja-ns.py

Friend Frank
Friend Anna

SOMEONE HAS A CAR: TRUE


In [2]:
```
<br/>

That's it. Namespaces in jinja.
