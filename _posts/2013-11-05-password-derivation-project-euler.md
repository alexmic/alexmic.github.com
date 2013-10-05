---
layout: post
title: "A solution to problem #79 on Project Euler"
tldr: "A simple solution to problem #79 using graph theory and breath-first searching."
published: true
---

<a href="http://projecteuler.net/problem=79">Problem 79</a> is an interesting problem which actually uncovers how easy it is for someone to recover the passcode of a user using a keylogger.

The problem is phrased as follows:

> A common security method used for online banking is to ask the user for three
> random characters from a passcode. For example, if the passcode was 531278,
> they may ask for the 2nd, 3rd, and 5th characters; the expected reply would be: 317.
>
> The text file, <a href="http://projecteuler.net/project/keylog.txt">keylog.txt</a>, contains fifty successful login attempts.
>
> Given that the three characters are always asked for in order, analyse the
> file so as to determine the shortest possible secret passcode of unknown     > length.

To solve the problem, we need to make one observation. Since we are after the **shortest** possible secret passcode, this means that each number (character) appearing in a logged attempt in `keylog.txt`, has to appear only **once** in the secret passcode. This holds because if a number were to appear twice, that would obviously not be the shortest solution possible.

The first step to the solution is to obtain the universe of numbers that should be in the passcode. Let's call this set **M**:

{% highlight python linenos=table %}
{% raw %}
def find_number_universe(keylog):
    numbers = set()
    for attempt in keylog:
        for num in attempt:
            numbers.add(num)
    return numbers
{% endraw %}
{% endhighlight %}

We then notice that the relationships between the numbers in each attempt form a graph. For example, '451' forms the graph described by this adjacency list:

{% highlight python linenos=table %}
{% raw %}
4 -> [5, 1],
5 -> [1]
{% endraw %}
{% endhighlight %}

Each X -> Y connection means *"X needs to be before Y in the passcode"*. We can then combine all the attempts and create one big graph that describes how the passcode numbers are laid out:

{% highlight python linenos=table %}
{% raw %}
def connections(attempt):
    l = len(attempt)
    for i in range(l - 1):
        for j in range(i + 1, l):
            yield attempt[i], attempt[j]


def make_number_graph(keylog):
    graph = defaultdict(set)
    for attempt in keylog:
        for a, b in connections(attempt):
            graph[a].add(b)
    return graph
{% endraw %}
{% endhighlight %}

The last bit of the puzzle is to traverse this graph and find the shortest path which should give us the shortest possible passcode. Since this graph is unweighted then a simple breadth-first search will do.

### Breadth-first searching

Breadth-first search is a method of traversing a graph by visiting the nodes in "breadth" than in "depth". That means, the algorithm traverses all the nodes of level N and then moves to the nodes of level N + 1. Typical implementations use a FIFO queue. Below is an animated image of the algorithm traversing a tree (source: <a href="http://en.wikipedia.org/wiki/Breadth-first_search">Wikipedia</a>).

<img class="centered" src="http://upload.wikimedia.org/wikipedia/commons/4/46/Animated_BFS.gif"/>

Our implementation follows a similar pattern and stops at the first match i.e returns the first path where the set difference with the number universe is empty:

{% highlight python linenos=table %}
{% raw %}
def find_smallest_code(start, graph, number_universe):
    queue = deque([(start, [start])])
    while queue:
        curr, path = queue.popleft()
        neighbours = graph.get(curr, [])
        for neighbour in neighbours:
            new_path = path + [neighbour]
            if not number_universe - set(new_path):
                return len(new_path), new_path
            queue.append((neighbour, new_path))
{% endraw %}
{% endhighlight %}

We now need to put everything together. Since we don't know a-priori which letter the passcode starts with, we need to run a search from each vertex and collect the results in a candidate list. The shortest code in that list is the answer:

{% highlight python linenos=table %}
{% raw %}
def solve(keylog):
    number_universe = find_number_universe(keylog)
    graph = make_number_graph(keylog)
    candidates = []
    for vertex in graph:
        code = find_smallest_code(vertex, graph, number_universe)
        if code: candidates.append(code)
    return sorted(candidates[0])
{% endraw %}
{% endhighlight %}

Running the code on keylog.txt returns **73162890** which succesfully solves the problem on Project Euler.