---
layout: post
title: Documenting your code with Sphinx, part 1
---

## Introduction 

The [Sphinx document
processor](http://www.pocoo.org/projects/sphinx/#sphinx) was
originally "was developed for the Python documentation and was
subsequently released as a separate tool under the BSD license" by the
geniuses at [pocoo](http://www.pocoo.org). Pocoo team and members are
also responsible for bringing the world the indispensible Pygments
library for highlighting all sorts of source code, the Flask web
"micro-framework", and two of Flask's important depenedencies,
[Werkzeug, a WSGI toolkit](http://werkzeug.pocoo.org/), and the [Jinja 2 template
engine](http://www.pocoo.org/projects/jinja2/#jinja2).

Here are a few really nice features of Sphinx, with more to come at a later date.
Or just read the [docs](http://sphinx-doc.org/index.html) yourself! They are 
good. I just have a hard time remembering certain useful things, like what I
wrote below!

### Automatic inclusion and formatting of source code documentation

First I want to show a
tool that allows us to use more readable [Google-style
docstrings](http://sphinxcontrib-napoleon.readthedocs.org/en/latest/example_google.html),
the Sphinx extension
[napoleon](sphinxcontrib-napoleon.readthedocs.org/en/latest/example_google.html).

With this you can structure your docstrings like in this example

{% highlight python %}
def how_funky(a1, a2):
    """How funky compares strings a1 and a2. 

    It tells you how absolutely funky they are and how relatively funky they 
    are to each other in a four-tuple, in that order. As you could guess then,
    relative funkiness is not commutative.

    Example:

        >>> dude = "rick james"
        >>> sonny = "sonny bono"
        >>> score = how_funky(dude, sonny)
        >>> # prints (87, 22, 113, 15)
        >>> print score
     
    Args:
        
        a1 (str): the first potentially funky string
        a2 (str): the second potentially funky string

    Returns:
        (int, int, int, int)

    Raises:
        TypeError: If a1 and a2 are not both strings, a TypeError is thrown
        because of Fermat's medium-sized theorem that says all numbers, and 
        Booleans are included, are equally funky.

    """
{% endhighlight %}

This docstring will be nicely formatted and printed when you call Sphinx's 
[autofunction](http://sphinx-doc.org/ext/autodoc.html?highlight=autofunction#directive-autofunction)
direction, and of course will be available in the interpreter when you run
`help(how_funky)`. 

You can put the `autofunction` directive anywhere in the ReST

{% highlight rst %}

Section 1
---------

You might not expect it, but all strings are not equally funky. Due to Fermat's
hard work, we know that all numeric values are equally funky. To calculate
the absolute and relative funkiness of two strings, we provide the 
``how_funky`` method

.. autofunction:: funky_module.how_funky

{% endhighlight %}

There are similar ["autodoc" functions for classes and
modules](http://sphinx-doc.org/ext/autodoc.html) that are very useful as well.

Do remember that any functions, classes, or modules, must be in the PATH
when building the documentation. This is done in `source/conf.py` with the 
command 

{% highlight python %}
sys.path.insert(0, os.path.abspath('/path/to/stuff'))
{% endhighlight %}

### Code highlighting

Sphinx can highlight all the [different languages that Pygments
can](http://pygments.org/docs/lexers/). To change the default highlighting
language to C, for example


{% highlight rst %}
.. highlight c
{% endhighlight %}

You can create code blocks like so:

{% highlight rst %}

.. code-block:: javascript

    var sq = function(x) { return x*x; };

{% endhighlight %}

Note that source code is indented once.

You can include whole files by using the `literalinclude` directive

{% highlight rst %}
.. literalinclude:: app.rb
    :language: ruby
    :emphasize-lines: 12,15-18

{% endhighlight %}

This will also add markup to emphasize the selected lines. This can also be used 
with code bocks. If you are changing the general markup parameters, you can
enable automatic line numbering for all code that exceeds a particular number
of lines. For example, to only add numbers when there are more than ten lines of
code to display

{% highlight rst %}
.. highlight:: python
    :linenothreshold: 10

{% endhighlight %}

To add line numbers to `any-code` block or `literalinclude`, just include the
`:linenos:` directive either by itself or along with other directives like
`emphasize-lines`.

{% highlight rst %}
.. code-block:: python
    :linenos:
    ...
{% endhighlight %}

### Inter-document referencing

Sphinx allows users to easily cross reference sections. To do this, use the 
general link syntax (`title of link <link-target>`) along with an extra
`:ref:` attached to the beginning, and no underscore after, like so:

{% highlight rst %}
:ref:`link title <link-target>`
{% endhighlight %}

The `link-target` needs be to be defined just above the section we want to 
reference, with the tagging done with exactly one line separating the 
tag and the section title:

{% highlight rst %}
.. _link-target:

Level two section header
-------------------------
{% endhighlight %}

Other objects can be referenced in the same way (images, code blocks, etc).

## Preview

Next time we'll cover:

- Documenting other Languages
- Exporting to other formats (PDF, eBook)
- Custom styling
