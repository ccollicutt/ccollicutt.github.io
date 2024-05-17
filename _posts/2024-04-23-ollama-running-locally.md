---
layout: post
title:  "Ollama & Llama 3:8b Running Locally"
categories:
header_image: "/img/ollama.png"
summary: "How to get a GPU and fine-tune an LLM with Unsloth"

---

# {{ page.title }}


First, we start up ollama.

```
$ ollama serve
```

OK, it's running.

```
$ curl localhost:11434 && echo
Ollama is running
```

Now, let's run Meta's Llama 3:8b.

> NOTE: This is only the 8b model, the smallest one so far, AFAIK.

```
$ ollama run llama3:8b
pulling manifest 
pulling 00e1317cbf74... 100% ▕█████████████████████████████████████████▏ 4.7 GB                         
pulling 4fa551d4f938... 100% ▕█████████████████████████████████████████▏  12 KB                         
pulling 8ab4849b038c... 100% ▕█████████████████████████████████████████▏  254 B                         
pulling c0aac7c7f00d... 100% ▕█████████████████████████████████████████▏  128 B                         
pulling db46ef36ef0b... 100% ▕█████████████████████████████████████████▏  483 B                         
verifying sha256 digest 
writing manifest 
removing any unused layers 
success 
>>> Send a message (/? for help)
```

Checkout memory usage. Not bad. Pretty small.

```
$ nvidia-smi --query-gpu=memory.used --format=csv,noheader,nounits -i 0
5308
```

## Intersecting Polygons

Lets ask Llama 3:8b to solve a problem: write the code to determine if two polygons are intersecting. This is a problem that some LLMs can't answer. (However, it is possible that Llama 3 has "learned" how to solve the problem from it being discussed recently on the internet, and given the fact that Llama 3 has new training data.)

Below, it looks like we've gotten a pretty good answer, as it uses the Python shapely library, which is the easiest thing to do, and what I'd expect to see.

```
>>> Write a function that calculates the intersection of two polygons. The function is given two polygons
...  represented as a list of points. A third polygon representing the intersection of the two is returne
... d, or an empty list of points if the intersection is empty.
Here's a Python function to calculate the intersection of two polygons:

import shapely.geometry
from typing import List

def polygon_intersection(polygon1: List[List[float]], polygon2: List[List[float]]) -> 
List[List[float]]:
    """
    Calculate the intersection of two polygons.

    Args:
    polygon1 (List[List[float]]): The first polygon represented as a list of points.
    polygon2 (List[List[float]]): The second polygon represented as a list of points.

    Returns:
    List[List[float]]: A list of points representing the intersection of the two polygons, or an 
empty list if the intersection is empty.
    """
    # Convert the input polygons to Shapely Polygon objects
    poly1 = shapely.geometry.Polygon(polygon1)
    poly2 = shapely.geometry.Polygon(polygon2)

    # Calculate the intersection polygon
    intersection = poly1.intersection(poly2)

    # If the intersection is not empty, return it as a list of points
    if not intersection.is_empty:
        return [(point.x, point.y) for point in intersection.exterior.coords]
    else:
        return []

This function uses the Shapely library to work with geometric shapes. It first converts the input 
polygons into Shapely Polygon objects. Then it calculates the intersection of these two polygons 
using the `intersection` method.

If the intersection is not empty, the function returns a list of points representing this polygon 
as a list of coordinates. If the intersection is empty, an empty list is returned.

Here's how you can use this function:

polygon1 = [(0, 0), (0, 2), (2, 2), (2, 0)]
polygon2 = [(1, 1), (1, 3), (3, 3), (3, 1)]

intersection_polygon = polygon_intersection(polygon1, polygon2)
print(intersection_polygon)

This will output the points that make up the intersection of the two polygons.
```

<br />

However, this code, as is, as provided, will not run.

```
$ python polygons.py 
Polygon 1:
(0, 0)
(0, 2)
(2, 2)
(2, 0)

Polygon 2:
(1, 1)
(1, 3)
(3, 3)
(3, 1)
Traceback (most recent call last):
  File "/home/curtis/working/intersecting-polygons/polygons.py", line 43, in <module>
    intersection_polygon = polygon_intersection(polygon1, polygon2)
                           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/curtis/working/intersecting-polygons/polygons.py", line 26, in polygon_intersection
    return [(point.x, point.y) for point in intersection.exterior.coords]
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/curtis/working/intersecting-polygons/polygons.py", line 26, in <listcomp>
    return [(point.x, point.y) for point in intersection.exterior.coords]
             ^^^^^^^
AttributeError: 'tuple' object has no attribute 'x'
```

<br/>

But I provide llama 3 the error and it gives me this response.

```
>>> I get this error: AttributeError: 'tuple' object has no attribute 'x'
That's a great error message!

It looks like the `point` variable is actually a tuple (a simple data structure in Python), not an 
object with an `x` attribute. This is because the `intersection.exterior.coords` returns a list of 
tuples, where each tuple represents a point on the polygon.

To fix this, you can simply change the line:

return [(point.x, point.y) for point in intersection.exterior.coords]

to:

return [tuple(point) for point in intersection.exterior.coords]

This will convert each tuple (representing a point) into a new tuple with the same elements.

Alternatively, you can also use a list comprehension to extract the `x` and `y` values from each 
tuple, like this:

return [[p[0], 
p[1][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p]p[1][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p]p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][p][
```

<br/>

> NOTE: Interesting additional output (the [p]s). I copied and pasted directly. Not sure what happened there...

I replaced that line of code as it suggested, and the program ran. For an 8b model this capability seems perfectly fine, at least to me, and was better than what I found ChatGPT 3.5 was capable of a few months ago, in that Llama 3 used the Shapely module, whereas ChatGPT 3.5 would not. I would imagine Llama 3:8b would fail if I told it not to use Shapely.

Results of running the provided code:

```
$ python polygons.py 
Polygon 1:
(0, 0)
(0, 2)
(2, 2)
(2, 0)

Polygon 2:
(1, 1)
(1, 3)
(3, 3)
(3, 1)

Intersection Polygon:
(2.0, 2.0)
(2.0, 1.0)
(1.0, 1.0)
(1.0, 2.0)
(2.0, 2.0)
```

<br/>

## ...But It Can’t Graph and Save an Image

Unfortunately, Llama 3:8b was unable to provide the code to plot the polygons and save them as an image file, which would have looked something like the below image. It may have been able to do this with a different prompt.

![img](/img/intersection.png)

## Overall: Impressive

This was a very quick test. I only spent a handful of minutes on it.

Llama 3:8B. It's fast. It can run locally. It's pretty "smart", although it would take a bit of manual configuration to get the code it output to work; I didn't give it much of a chance to get things right. Overall, I'm impressed with this little LLM--its compressed a lot of information. 