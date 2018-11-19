---
layout: post
title: Inverting, Reversing, or Mirroring a Binary Tree
categories:

---

# {{ page.title }}

Interviews based on algorithms and data structures will continue to be the norm. Here I take a look at inverting a binary tree.

## tldr

Reversing/mirroring/inverting, whatever you call it, has a nice recursive answer.

```
def recurseInvertTree(root):
    if root is not None:
        root.left, root.right = invertTree(root.right), \
                                invertTree(root.left)

    return root  
```

<br />

That's it. :)

## History

The way organizations hire developers is varied. However, **for better or worse**, I feel that whiteboarding interviews that involve solving complex algorithm based problems is the default model, and will continue to be the default model. Some people like it, some don't, but regardless it's the defacto standard, and it's unavoidable.

One of the more famous kurfuffles around algorithmic whiteboarding interviews came out of this tweet:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Google: 90% of our engineers use the software you wrote (Homebrew), but you can’t invert a binary tree on a whiteboard so fuck off.</p>&mdash; Max Howell (@mxcl) <a href="https://twitter.com/mxcl/status/608682016205344768?ref_src=twsrc%5Etfw">June 10, 2015</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The developer of a popular and useful open source project, Homebrew, did not pass a whiteboarding interview in which he was apparently asked to invert a binary tree on a whiteboard.

Despite the drama around this particular tweet, being able to perform well in a whiteboarding interview could mean the difference between getting an Okay job versus getting a great job, so it's a skill worth having. Over a career it could mean hundreds of thousands of dollars in salary.

So how can we invert/reverse/mirror a binary tree?

## Naming Things

The first problem I ran into when researching this was what to call it. Inverting seems like the most common name, but reversing or mirroring would work too.

For the purposes of this blog post, I'll use inverting. I think an example is easiest to explain.

Original tree. (Sorry about printing it like this, I actually prefer it as you don't run out of horizontal space as easily.)

```
            14
        6
            13
    2
            12
        5
            11
0
            10
        4
            9
    1
            8
        3
            7
```

Inverted tree.

```
            7
        3
            8
    1
            9
        4
            10
0
            11
        5
            12
    2
            13
        6
            14
```

It's the same tree but the leafs have been flipped/reversed what have you.

*Aside: it's kind of amazing how many names there could be for this. Inverted. Reversed. Flipped. Mirrored. Transposed. Part of answering an interview question like this would be getting some terms defined.*

## Recursion

I believe the reason that this question was originally asked is because it has such a nice recursive answer. I can imagine interviewers asking the question, then watching a developer muddle through it for X minutes, getting an answer, and then being shown that there is a ~4 line solution using recursion.

```
def recurseInvertTree(root):
    if root is not None:
        root.left, root.right = invertTree(root.right), \
                                invertTree(root.left)

    return root  
```

I mean, ultimately, that is a pretty easy algorithm to whiteboard.

But there are other ways to solve this as well.

## Depth First Search Stack solution

This doesn't seem too bad either.

```
def stackInvertTree(root):
    if root is not None:
        nodes = []
        nodes.append(root)
        while nodes:
            node = nodes.pop()
            node.left, node.right = node.right, node.left
            if node.left is not None:
                nodes.append(node.left)
            if node.right is not None:
                nodes.append(node.right)

    return root
```

I'm guessing that is what a lot of people would come up with if the recursive version wasn't obvious to them.

## Breadth First Search with Queue

We can use a queue as well.

```
def queueInvertTree(root):
    queue = collections.deque([(root)])
    while queue:
        node = queue.popleft()
        if node:
            node.left, node.right = node.right, node.left
            queue.append(node.left)
            queue.append(node.right)
    return root
```   

That's nice too.

## Conclusion

These are the most common solutions I can find. There is a lot of computer science in this question, and I'm only touching the layperson's surface here. Much more to discover.

If you see any issues with what I've put up here, likely around naming, do let me know.