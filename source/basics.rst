.. highlight:: none
    :linenothreshold: 1

A Basic Example
===============

The first thing you'll need is an XML document.  The example programs in this
section will use the :download:`playlist.xml </code/playlist.xml>`
file shown below. This file contains details of five different movies:

.. literalinclude:: /code/playlist.xml
    :language: xml
    :linenos:

.. note::

    Although this XML document contains details which came from the fabulous
    `IMDb.com <http://www.imdb.com/>`_ web site, the file structure was created
    specifically for this example and does not represent an actual API for
    querying movie details.

Once you have the sample XML document, you can use :download:`this script
</code/010-list-titles.pl>` to extract and print the title of each movie,
in the order they appear in the XML:

.. literalinclude:: /code/010-list-titles.pl
    :language: perl

and will produce the following output:

.. literalinclude:: /_output/010-list-titles.pl-out
    :language: none

.. sidebar:: Is XML::LibXML installed?

    If you try running this example script but you don't have the
    ``XML::LibXML`` module installed on your system, then you'll get an error
    like this:

        Can't locate XML/LibXML.pm in @INC ... at ./010-list-titles.pl line 7.

    If you do get this error, then refer to :doc:`installation` for help on
    installing ``XML::LibXML``.

If we break the example down line-by-line we see that after a standard
boilerplate section, the script loads the ``XML::LibXML`` module:

.. literalinclude:: /code/010-list-titles.pl
    :language: perl
    :lines: 7

Next, the ``load_xml()`` class method is called to parse the XML file and
return a document object:

.. literalinclude:: /code/010-list-titles.pl
    :language: perl
    :lines: 11

The ``$dom`` variable now contains an object representing all the elements of
the XML document arranged in a tree structure known as a
:doc:`Document Object Model <dom>` or 'DOM'.

Finally we get to the guts of the script where the ``findnodes()`` method is
called to search the DOM for the elements we're interested in and a ``foreach``
loop is used to iterate through the matching elements:

.. literalinclude:: /code/010-list-titles.pl
    :language: perl
    :lines: 13-15

The ``findnodes()`` method takes one argument - an **XPath expression**.  This
is a string describing the location and characteristics of the elements we want
to find.  XPath is a query language and the way we use it to select elements
from the DOM is similar to the way we use SQL to select records from a
relational database.  The next section (:doc:`xpath`) will include examples of
more complex queries.

The ``findnodes()`` method returns a list of objects from the DOM that match
the XPath expression. Each time through the loop, ``$title`` will contain an
object representing the next matching element.  This object provides a number
of properties and methods that you can use to access the element and its
attributes, as well as any text content and 'child' elements.

Inside the loop, this example simply calls the ``to_literal()`` method to get
the text content of the element.  The string returned by ``to_literal()`` will
not include any of the attributes but will include the text content of any
child elements.

Other XML sources
-----------------

The first example script called ``XML::LibXML->load_xml()`` with the
``location`` argument set to the name of a file.  The ``location`` argument
also accepts a URL:

.. code-block:: perl

    $dom = XML::LibXML->load_xml(location => 'http://techcrunch.com/feed/');

If you have the XML in a string, instead of ``location``, use ``string``:

.. code-block:: perl

    $dom = XML::LibXML->load_xml(string => $xml_string);

Or, you can provide a Perl file handle to parse from an  open file or socket,
using ``IO``:

.. code-block:: perl

    $dom = XML::LibXML->load_xml(IO => $fh);

When providing a string or a file handle, it's crucial that you **do not**
decode the bytes of the source data (for example by using ``':utf8'`` when
opening a file).  The underlying ``libxml2`` library is written in C to decode
bytes and does not understand Perl's character strings.  If you have assembled
your XML document by concatenating Perl character strings, you will need to
encode it to a byte string (for example using ``Encode::encode_utf8()``) and
then pass the byte string to the parser.

If you have enabled UTF-8 globally with something like this in your script:

.. code-block:: perl

    use open ':encoding(utf8)';

Then you'll need to turn **off** the encoding IO layers for any file handle
that you pass to XML::LibXML:

.. code-block:: perl

    open my $fh, '<', $filename;
    binmode $fh, ':raw';
    $dom = XML::LibXML->load_xml(IO => $fh);

A more complex example
----------------------

Now let's look at a slightly more complex example.  :download:`This script
</code/040-movie-details.pl>` takes the same XML input and extracts more
details from each ``<movie>`` element:

.. literalinclude:: /code/040-movie-details.pl
    :language: perl

and will produce the following output:

.. literalinclude:: /_output/040-movie-details.pl-out
    :language: none

Let's compare the main loop of the first script:

.. literalinclude:: /code/010-list-titles.pl
    :language: perl
    :lines: 13-15

with the main loop of the second script:

.. literalinclude:: /code/040-movie-details.pl
    :language: perl
    :lines: 13-23

The structure of the main loop is very similar but the XPath expression
passed to ``findnodes()`` is different in each case:

``'/playlist/movie/title'``
    | Will match every ``<title>`` element which is the child of ...
    | a ``<movie>`` element which is the child of ...
    | a ``<playlist>`` element which is ...
    | the top-level element in the document.

    Or, to phrase it a different way, the search will start at the top of the
    document and look for a ``<playlist>`` element; if one is found, the search
    will continue for child ``<movie>`` elements; and for each one that is
    found the search will continue for child ``<title>`` elements.

``'//movie'``
    Will match every ``<movie>`` element at any level of nesting.

In both cases, the XPath expression starts with a '/' which means the search
will start at the the top of the document.

Inside the second script's loop are a number of calls to ``findvalue()``.  This
is a handy shortcut method that is typically used when you expect the XPath
expression to match *exactly one node*.  It combines the functionality of
``findnodes()`` and ``to_literal()`` into a single method.  So this code:

.. code-block:: perl

    $movie->findvalue('./title');

is equivalent to:

.. code-block:: perl

    $movie->findnodes('./title')->to_literal();

There are a couple of other interesting differences with the XPath searches in
the loop compared to previous examples.  Firstly, the ``findvalue()`` method is
being called on ``$movie`` (which represents one ``<movie>`` element) rather
than on ``$dom`` (which represents the whole document). This means that the
``$movie`` element is the **context element**.  Secondly, the XPath expression
starts with a '.' which means: start the search at the context element rather
than at the top of the document.

This second script illustrates a common pattern when working with ``XML::LibXML``:

#. find 'interesting' elements using an XPath query starting with '/' or '//'

#. iterate through those elements in a ``foreach`` loop

#. get additional data from child elements using XPath queries starting with '.'


Accessing attributes
--------------------

When listing cast members in the main loop of the script above, this code ...

.. literalinclude:: /code/040-movie-details.pl
    :language: perl
    :lines: 18-21

is used to transform this XML ...

.. code-block:: xml
    :linenos:

    <cast>
      <person name="Matt Damon" role="Mark Watney" />
      <person name="Jessica Chastain" role="Melissa Lewis" />
      <person name="Kristen Wiig" role="Annie Montrose" />
    </cast>

into this output:

.. literalinclude:: /_output/040-movie-details.pl-out
    :language: none
    :lines: 29

In an XPath expression, a name that starts with ``@`` will match an attribute
rather than an element, so ``'person/@name'`` refers to an attribute called
``name`` on a ``<person>`` element.  In this case, the call to
``findnodes('./cast/person/@name')`` will return three DOM nodes representing
attribute values which are then transformed into plain strings using
``to_literal()``, as we've seen for element nodes, inside a `map
<http://perldoc.perl.org/functions/map.html>`_ block.

Another approach is to select the *element* with XPath and then call a DOM
method on the element node to get the attribute value:

.. literalinclude:: /code/050-attributes.pl
    :language: perl
    :lines: 31-34

.. _tied-attribute-hash:

Attributes via tied hash
------------------------

There's a shortcut syntax you can use to make this even easier, simply treat
the element node as a hashref:

.. literalinclude:: /code/050-attributes.pl
    :language: perl
    :lines: 42-45

You might be a bit wary of poking around directly inside the element object,
rather than using accessor methods. But don't worry, that's **not** what this
shortcut syntax is doing.  Instead, every `XML::LibXML::Element
<https://metacpan.org/pod/XML::LibXML::Element>`_ object returned from the
XPath query has been `'tied'
<https://metacpan.org/pod/distribution/perl/pod/perltie.pod>`_ using
`XML::LibXML::AttributeHash
<https://metacpan.org/pod/XML::LibXML::AttributeHash>`_ so that hash lookups
'inside' the object actually get proxied to ``getAttribute()`` method calls.

This technique is less efficient than calling ``getAttribute()`` directly but
it is very convenient when you want to access more than one attribute of an
element or when you want to interpolate an attribute value into a string:

.. literalinclude:: /code/050-attributes.pl
    :language: perl
    :lines: 53-56

Which will produce this output:

.. literalinclude:: /_output/050-attributes.pl-out
    :language: none
    :lines: 5-8

.. note::

  Overloading 'Element' nodes to support tied hash access to attribute values
  was added in version 1.91 of XML::LibXML.  If the examples above don't work
  for you then it may be because you have a very old version installed.

That's it for the basic examples.  The next topic will look more closely at
:doc:`XPath expressions <xpath>`.
