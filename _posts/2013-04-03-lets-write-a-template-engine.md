---
layout: post
title: 'Exercise: Building a toy template engine in Python.'
draft: true
---
{% highlight python %}
def split_children(self):
    if_branch, else_branch = [], []
    in_if_branch = True
    for child in self.children:
        if isinstance(child, _Else):
            in_if_branch = False
        else:
            if in_if_branch:
                if_branch.append(child)
            else:
                else_branch.append(child)
    return if_branch, else_branch
{% endhighlight %}