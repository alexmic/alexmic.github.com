---
layout: post
title: "Approach: Building a toy template engine in Python"
tldr: "Let's build a simplistic template engine and explore how it works under the hood – in case you were ever wondering."
published: true
---

If you wish to dive straight into the code, then [Github](https://github.com/alexmic/microtemplates) is your friend.

### Language design

Our language is pretty basic. We will have two types of tags, *variables* and *blocks*.

{% highlight html linenos=table %}
{% raw %}
<!-- variables are surrounded by `{{` and `}}` -->
<div>{{my_var}}</div>

<!-- blocks are surrounded by `{%` and `%}` -->
{% each items %}
    <div>{{it}}</div>
{% end %}
{% endraw %}
{% endhighlight %}

Most blocks need to be *closed* as in the example and the closing tag is just the
word *end*.

Our template engine will be able to handle basic loops and conditionals. We will
also add support for callables in blocks – personally, I find it handy being
able to call arbitrary Python functions in my templates.

#### Loops

Loops allow for iterations over collections or iterable objects.

{% highlight html linenos=table %}
{% raw %}
{% each people %}
    <div>{{it.name}}</div>
{% end %}

{% each [1, 2, 3] %}
    <div>{{it}}</div>
{% end %}

{% each records %}
    <div>{{..name}}</div>
{% end %}
{% endraw %}
{% endhighlight %}

In the example above, *people* is the collection and *it* refers to the current
item in the iteration. Dotted paths in names will resolve to nested dictionary
attributes. Using *'..'* we can access names in the parent context.

#### Conditionals

Conditionals need no explanation. Our language will support *if* and *else* constructs,
and the following operators: *==, <=, >=, !=, is, >, <*.

{% highlight html linenos=table %}
{% raw %}
{% if num > 5 %}
    <div>more than 5</div>
{% else %}
    <div>less than or equal to 5</div>
{% end %}
{% endraw %}
{% endhighlight %}

#### Callables

Callables can be passed via the template context and get called with positional
or keyword arguments in the template. Call blocks do not need to be closed.

{% highlight html linenos=table %}
{% raw %}
<!-- supports positional arguments... -->
<div class='date'>{% call prettify date_created %}</div>
<!-- ...and keyword arguments -->
<div>{% call log 'here' verbosity='debug' %}</div>
{% endraw %}
{% endhighlight %}

### Theory

Before delving into the details of how our engine will compile and render the templates
we need to talk a bit about how we will represent a compiled template in memory.

Compilers use [Abstract Syntax Trees](http://en.wikipedia.org/wiki/Abstract_syntax_tree)
to represent the structure of a computer program. ASTs are the product of lexical analysis on the source code.
An AST has a lot of advantages over source code since it does not contain unnecessary
textual elements such as delimiters; furthermore, nodes in the tree can be enhanced with
attributes without altering the actual code.

We will parse and analyse the template and build such a tree to represent the compiled template.
To render the template we will walk the tree, pass the correct context into each node and spit
out HTML.

### Tokenizing the template

The first step in parsing the template is splitting the contents into fragments. Each
fragment can either be arbitrary HTML or a tag. To split the contents we will
use [regular expressions](http://docs.python.org/2/library/re.html) and the *split()*
function.

{% highlight python linenos=table %}
{% raw %}
VAR_TOKEN_START = '{{'
VAR_TOKEN_END = '}}'
BLOCK_TOKEN_START = '{%'
BLOCK_TOKEN_END = '%}'
TOK_REGEX = re.compile(r"(%s.*?%s|%s.*?%s)" % (
    VAR_TOKEN_START,
    VAR_TOKEN_END,
    BLOCK_TOKEN_START,
    BLOCK_TOKEN_END
))
{% endraw %}
{% endhighlight %}

Let's analyze *TOK_REGEX* a bit. We can see that there's an option between
a variable tag and a block tag. That makes sense – we want to split on either
variables or blocks. We surround the option with capturing parenthesis to include
the matching text in the resulting tokens. The *?* inside the options is for
*non-greedy* repetition. We want our regex to be lazy and stop on first match,
so, for example, we can extract variables inside blocks. [This](http://www.regular-expressions.info/repeat.html)
is a very nice explanation of how to control the greediness of regular expressions.

The following example shows our regular expression in action:

{% highlight python linenos=table %}
{% raw %}
>>> TOK_REGEX.split('{% each vars %}<i>{{it}}</i>{% endeach %}')
['{% each vars %}', '<i>', '{{it}}', '</i>', '{% endeach %}']
{% endraw %}
{% endhighlight %}

We will encapsulate each fragment of text in a *Fragment* object. This object
will determine the fragment type and prepare the fragment for consumption by
the compile function. Fragments can be one of four types:

{% highlight python linenos=table %}
VAR_FRAGMENT = 0
OPEN_BLOCK_FRAGMENT = 1
CLOSE_BLOCK_FRAGMENT = 2
TEXT_FRAGMENT = 3
{% endhighlight %}


### Building the AST

Once we've tokenized the template content it's time to go through each fragment and build
the syntax tree. We will use a *Node* class to serve as the base class for tree nodes, and
then create concrete subclasses for each possible node type. A subclass
should provide implementations for *process_fragment()* and *render()*.
*process_fragment()* is used to further parse the fragment contents and store
necessary attributes on the *Node* object. *render()* is responsible for converting
the node to HTML using the provided context.

A subclass could optionally implement *enter_scope()* and *exit_scope()* which
are hooks that get called by the compiler during compilation and should perform
further initialization and cleanup. *enter_scope()* is called when the node
creates a new scope (more on this later), and *exit_scope()* is called when the node's
scope is about to get popped off the scope stack.

This is our base *Node* class:

{% highlight python linenos=table %}
class _Node(object):
    def __init__(self, fragment=None):
        self.children = []
        self.creates_scope = False
        self.process_fragment(fragment)

    def process_fragment(self, fragment):
        pass

    def enter_scope(self):
        pass

    def render(self, context):
        pass

    def exit_scope(self):
        pass

    def render_children(self, context, children=None):
        if children is None:
            children = self.children
        def render_child(child):
            child_html = child.render(context)
            return '' if not child_html else str(child_html)
        return ''.join(map(render_child, children))
{% endhighlight %}

As an example of a concrete subclass, this is a *Variable* node:

{% highlight python linenos=table %}
class _Variable(_Node):
    def process_fragment(self, fragment):
        self.name = fragment

    def render(self, context):
        return resolve_in_context(self.name, context)
{% endhighlight %}

In order to determine the node type (and hence instantiate the correct node class) we will look at the
fragment type and text. *Text* and *Variable* fragments directly translate to text nodes and variable nodes.
*Block* fragments need a bit more processing – their type is denoted by the
*block command* which is the first word in the fragment text. For example, the fragment:
{% highlight python linenos=table %}
{% raw %}
{% each items %}
{% endraw %}
{% endhighlight %}
is a block node of *'each'* type since the block command is *each*.

A node can also create a scope. During compiling, we keep track of the current
scope and new nodes are added as children of that scope. Once we encounter a correct
closing tag we close the scope, pop it off the scope stack and set the top of the stack
as the new current scope.

{% highlight python linenos=table %}
def compile(self):
    root = _Root()
    scope_stack = [root]
    for fragment in self.each_fragment():
        if not scope_stack:
            raise TemplateError('nesting issues')
        parent_scope = scope_stack[-1]
        if fragment.type == CLOSE_BLOCK_FRAGMENT:
            parent_scope.exit_scope()
            scope_stack.pop()
            continue
        new_node = self.create_node(fragment)
        if new_node:
            parent_scope.children.append(new_node)
            if new_node.creates_scope:
                scope_stack.append(new_node)
                new_node.enter_scope()
    return root
{% endhighlight %}

### Rendering

The last step in the pipeline is to render the AST to HTML. We visit all the
nodes in the AST and call *render()* passing as a parameter the context given
to the template. During rendering we will need to deduce whether we are dealing
with literals or context variable names that need to be resolved. For this, we will use
[ast.literal_eval()](http://docs.python.org/2/library/ast.html#ast.literal%5Feval) which
can safely evaluate strings containing Python code:

{% highlight python linenos=table %}
def eval_expression(expr):
    try:
        return 'literal', ast.literal_eval(expr)
    except ValueError, SyntaxError:
        return 'name', expr
{% endhighlight %}

If we are dealing with a context variable name instead of a literal then we need
to resolve it by searching for its value in the context. We need to take care of dotted names
and names that reference the parent context. Here's our resolving function which is
the final piece of the puzzle:

{% highlight python linenos=table %}
def resolve(name, context):
    if name.startswith('..'):
        context = context.get('..', {})
        name = name[2:]
    try:
        for tok in name.split('.'):
            context = context[tok]
        return context
    except KeyError:
        raise TemplateContextError(name)
{% endhighlight %}


### Conclusion

I hope this academic exercise gave you a taste of how the internals of a templating engine
might work. This is far and away from production quality but it can serve as the
basis of better things.

You can find the full code on [Github](https://github.com/alexmic/microtemplates), and
you can discuss this further on <a class="hn" target="_blank" href="https://news.ycombinator.com/item?id=5535040">Hacker News</a>.

*Huge thanks* to Nassos Hadjipapas, Alex Loizou, Panagiotis Papageorgiou and Gearoid O'Rourke for reviewing.