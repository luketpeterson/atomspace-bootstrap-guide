.. role:: scheme(code)
   :language: scheme

.. _01_getting_oriented:

========================================================================
Getting Oriented & Basic Concepts
========================================================================

For me, the easiest way to begin to conceptualize the atomspace is as a database of *Atoms*.  So each Atom is a unique record.  And each atom can store a table of *Values*, where a key can access the value associated with the atom.
Unlike some database systems, e.g. SQL, Atomspace doesn't require a rigidly defined table.  Each atom may have its own individual keys and values.  In that sense, the Atomspace is more like some of the newer NoSQL databases.

More info about atoms is here: `<https://wiki.opencog.org/w/Atom>`_

More info about values is here: `<https://wiki.opencog.org/w/Value>`_

I'll start out talking about the two most fundamental flavors of atoms: *Nodes* and *Links*.

A node is uniquely identified by a combination of its *Name* & *Type*.  I like to think of nodes as nouns.  This isn't a perfect metaphor, but it'll do for now.

Links exist to reference other atoms. A link is uniquely identified by
its type & *Outbound Set*, which is the set of all other atoms the link references.  The atomspace is a *hypergraph*,
meaning that links may also reference other links, in addition to referencing nodes.

Links come in both ordered and unordered flavors.

- An unordered link might be used to define a set, like the set of all animals.
- An ordered link may be used to define a list or sequence, like the colors of the rainbow.
- An ordered link with only two referants can be used to define a binary relationship, like "A cat is an animal."  In other words, ordered links can express a direction; The statement: "A cat is an animal." is not equivalent to "An animal is a cat.".  Directionality can be important.

A more complete overview of the atoms can be found here: `<https://wiki.opencog.org/w/Atom_types>`_  But I didn't initially have the needed context to understand the information on that page until I'd spent a bit more time with the Atomspace.
I'd recommend wading into the shallow-end of the pool, especially if you're unfamiliar with *Knowledge Base* concepts.

So, Let's get started.

Installation & Getting to the Scheme Shell
------------------------------------------------------------------------

You have many options getting OpenCog set up, and they are well documented at: `<https://wiki.opencog.org/w/Building_OpenCog>`_

Personally, I found the fastest route was to use the Docker containers.  The procedure is documented here: `<https://github.com/opencog/docker/blob/master/opencog/README.md>`_

If Docker is unfamiliar, this blog helped me through my Docker-specific conceptual issues.  `<https://blog.hipolabs.com/understanding-docker-without-losing-your-shit-cf2b30307c63>`_

.. note:: If you're installing OpenCog in Docker on a recent Mac, you'll need to fix the reference to :code:`/etc/localtime/`.  See this issue on Github for the work-around I used.  `<https://github.com/opencog/docker/issues/151>`_

Once you have the OpenCog container running, you can now get to the interactive Scheme shell.  You have multiple options that both get you to basically the same place.

Option 1, Run Guile in the Docker terminal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If you started the OpenCog container using :code:`docker run` or a similar command, such as :code:`docker-compose run --service-ports dev`, your terminal should already be running in a shell within the container.
You can verify this by executing :code:`whoami`, and verifying that your user is now :code:`opencog`.

If you aren't already in the container's shell, you can launch a new shell using the *CLI* button in the Docker Dashboard GUI, or by running :code:`docker attach` from another terminal.

Now, you can run :code:`guile`, and be at an interactive Scheme interpreter prompt.  However, I'd recommend configuring your :code:`~/.guile` file first to automatically load the OpenCog modules and make the shell more ergonomic to use.  Instructions are here: `<https://wiki.opencog.org/w/Scheme#The_guile_REPL_shell>`_

Option 2, Telnet into the OpenCog Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The other option is to run :code:`cogserver` on the OpenCog virtual machine, and then connect using :code:`telnet localhost 17001`

Once connected to the cogserver, you can access the Scheme shell by running the :code:`scm` command.

The documentation asserts that the cogserver approach is superior for performance, but at my novice level, I find Option 1 to be more convenient.  Also I enjoy the customizations I made to my `~/.guile` file, but that file isn't loaded when guile is run by the OpenCog server process.

Poking around in Scheme
------------------------------------------------------------------------
Scheme is a full-featured programming language.  For example, you can try to execute the following:

.. code-block:: scheme

   (display "Hello, World!\n")

You can read more about Scheme at: `<http://www.shido.info/lisp/idx_scm_e.html>`_ among many other places.  Within the OpenCog Scheme shell you can use *cons*, *car* and *cdr* 'til the cows come home.
I often find it handy to have the Scheme environment right at my fingertips.

But we're here in a Scheme shell because Scheme is a (**the??**) first-class interface to the atomspace.  
However it's important to point out that the Atomspace and the scheme language are conceptually distinct software components.  Neither one strictly relies on the other.  Scheme is an interface to the Atomspace, but the Atomspace isn't "Running in Scheme".

Furthermore, irrespective of Scheme, the Atomspace itself can also behave as a Turing-complete program execution environment for Atomese, the language of the Atomspace.
I found this a bit confusing because it means there are at least two ways to do most things within an OpenCog Scheme shell.
However, the point of this guide is to focus on the atomspace, so I'll do my best to keep the Scheme stuff to a minimum from here onward.

Creating some Node Atoms
------------------------------------------------------------------------

Executing this statement will create a new atom.  A new node of type: ConceptNode that has the Name "Dog"

.. code-block:: scheme

   (ConceptNode "Dog")

The :code:`ConceptNode` Scheme function above invokes the :code:`cog-new-node` native Atomspace function.  So the above code is equivalent to:

.. code-block:: scheme

   (cog-new-node 'ConceptNode "Dog")

The single quote in :code:`'ConceptNode` above tells the Scheme interpreter that :code:`ConceptNode` should be treated as a literal symbol, and not to attempt to evaluate it further.

If you just ran the code above, the :code:`cog-new-node` function would have retrieved the first node you created, rather than creating a duplicate.  A node is defined as a unique Name & Type combination, so duplicates cannot exist, by definition.

Sometimes it's handy to use a Scheme symbol to refer to an atom.  This is done with Scheme's :scheme:`define` feature, like this:

.. code-block:: scheme

   (define dog_concept (ConceptNode "Dog"))

Also, you can print an atom using Scheme's :scheme:`display` function.  If the atom is a link, it will recursively display the referenced atoms as well.

.. code-block:: scheme

   (display dog_concept)

Now, you can see all the atoms in the Atomspace with this function:

.. code-block:: scheme

   (cog-prt-atomspace)

At the moment, there is probably just one atom in your Atomspace, unless you've gone off-script already :-)  Eventually, the output of the :code:`cog-prt-atomspace` function might get a little too verbose to use unfiltered, as we create more and more atoms.

.. note:: Later on, we'll create multiple Atomspaces.  This will be useful for segmentation of the knowledge base.  Temporary child Atomspaces may be instantiated on a stack, to facilitate things like counterfactual or hypothetical reasoning.  These are more advanced topics, however, and it'll be a while before we get there.

.. note:: QUESTION for someone smarter than me. Does the sub-atomspace management allow for conflicting assertions, so long as the two child atomspaces aren't active simultaneously?  For example, can the Newtonian and the Einsteinian equations of gravity be expressed in the same KB using different child atomspaces?

Values associated with Atoms
------------------------------------------------------------------------

We can attach values to atoms.  The below snippet will create a concept node for a particular dog, "Fido the Dog", and another one for the concept of "weight_in_kg".  Then it will assign Fido a weight value.

.. note:: As you can see below, we've abbreviated :code:`ConceptNode` as just :code:`Concept`.  Abbreviations like this are common in the Atomspace.

.. code-block:: scheme

   (cog-set-value! (Concept "Fido the Dog")
      (Concept "weight_in_kg") (FloatValue 12.5))

As you can see above, the "weight_in_kg" key is actually a separate :code:`ConceptNode` atom.  Any atom can be used as the key to a value associated with another atom.

However, convention dictates that a :code:`PredicateNode`, which can be abbreviated as just :code:`Predicate`, is the best type of atom to use as a key.
A :code:`PredicateNode` is a special type of node used to define certain formal relationships.  There is a lot to say about :code:`PredicateNode` atoms, and we'll cover them in depth during the next chapters.
You can skip ahead if you want.  :ref:`Structured Knowledge <02_representing_knowledge>`

So here is the above example, with a :code:`PredicateNode` instead of a :code:`ConceptNode` as the key:

.. code-block:: scheme

   (cog-set-value! (Concept "Fido the Dog")
      (Predicate "weight_in_kg") (FloatValue 12.5))

Values can also be non-numerical.  Here's an example string value for Fido.

.. code-block:: scheme

   (cog-set-value! (Concept "Fido the Dog")
      (Predicate "nickname") (StringValue "Good Boy"))

You can access values on an atom using the :code:`cog-value` function, like this:

.. code-block:: scheme

   (cog-value (Concept "Fido the Dog")
      (Predicate "weight_in_kg"))

The above snippet will return an Atomspace :code:`FloatValue`, which you can turn into a scheme list using :code:`cog-value->list` like this:

.. code-block:: scheme

   (cog-value->list
      (cog-value (Concept "Fido the Dog")
         (Predicate "weight_in_kg")))

And now you can do stuff in Scheme with the result.  You may have noticed that the value was returned as a Scheme list, as opposed to another Scheme primitive data type, such as a Scheme number.
All Atomspace values are arrays of elements.  So a single number is represented a list of numerical values with a list-length of one.
Throughout the other OpenCog documentation, you will often see values being initialized with a list of multiple elements.

Because values are returned to Scheme as a list, I need to use :code:`car` to access the first element.  This example uses Scheme to add 50 to Fido's weight:

.. code-block:: scheme

   (+
      (car
         (cog-value->list
            (cog-value (Concept "Fido the Dog")
               (Predicate "weight_in_kg"))))
      50
   )

But I wasn't supposed to focus on Scheme.  Sorry about that.  The Atomspace has its own way to express arithmetic operations, and we'll get to that soon enough.  I mainly wanted to demonstrate the way the :code:`cog-value->list` function could be used as a bridge to get data from the Atomspace back into Scheme.

If you know the index of the value element you are interested in, you can use the :code:`cog-value-ref` function.  This is cleaner and more efficient than using Scheme's list element access semantics, (e.g. :scheme:`car`, :scheme:`cdr`, :scheme:`list-ref`, etc.).  It's also very handy if you know you have a 1-element value, such as in our example.

.. code-block:: scheme

   (+
      (cog-value-ref
         (cog-value (Concept "Fido the Dog")
            (Predicate "weight_in_kg"))
         0)
      50
   )

Additionally, you can retrieve the complete set of keys associated with an atom using the :code:`cog-keys` function.  Here is an example:

.. code-block:: scheme

   (cog-keys (Concept "Fido the Dog"))

And if you want to list both the keys and values, you can do it with :code:`cog-keys->alist` like this:

.. code-block:: scheme

   (cog-keys->alist (ConceptNode "Fido the Dog"))

And, should you want to, you can remove a key-value pair from an atom entirely by setting they key to either :code:`#f` (The symbol for 'false') or :code:`'()` (The empty list) using the :code:`cog-set-value!` function.  For example, you could use the snippet below to delete the "nickname" we assigned earlier.

.. code-block:: scheme

   (cog-set-value! (Concept "Fido the Dog")
      (Predicate "nickname") #f)

More about Values 
------------------------------------------------------------------------

In addition to Floats and Strings, references to other Atoms can be used as values.  Here is an example.

.. code-block:: scheme

   (cog-set-value! (Concept "Fido the Dog")
      (Predicate "favorite_toy") (Concept "Squeaky Ball"))

In the Atomspace, every atom can be used as a value, but not every value can be used as an atom.

Other value types are described on the OpenCog wiki `here <https://wiki.opencog.org/w/Value>`_.  We'll get to *TruthValue* in the near future but it deserves a whole section.

Right now, let's talk about :code:`LinkValue`.

In my mind, it is unfortunate that :code:`LinkValue` and Link have a name collision, as I believe they have some major conceptual differences.  But c'est la vie.
A :code:`LinkValue` is essentially a list.  But aren't all values lists?  Well yes, but :code:`LinkValue` can be used to compose a heterogeneous list of different value types.
So, while a :code:`FloatValue` is a list of floats and a :code:`StringValue` is a list of Strings, a :code:`LinkValue` is a list of abstract values.

Below is an example lifted right out of the documentation for :code:`LinkValue`, which is here: `<https://wiki.opencog.org/w/LinkValue>`_

.. code-block:: scheme

   (cog-set-value! (Concept "abc") (Predicate "key")
   (LinkValue 
      (StringValue "foo") 
      (FloatValue 41 43 43 44)
      (Concept "foobar")))

.. note:: QUESTION for someone smarter than me. If :code:`LinkValue` is an array, Why isn't there a fundamental "MapValue" type with its own internal keys?
   The Value documentation mentions the :code:`LinkValue` being useful for structured data, e.g. from JSON or YAML, but without a dictionary or map type, it's not complete.
   I understand maps can be created by embedded atoms for each sub-dictionary, and using their key-value spaces, but I thought one of the features of Values was that they have lower overhead than Atoms, so creating embedded atoms seems to defeat that.

In summary, here are some of the conceptual differences between Atoms and Values.

============================================   ============================================
   Atoms                                          Values
============================================   ============================================
   All Atoms reside in the Atomspace              Values can live anywhere
   Atoms persist until they are deleted           Values go away when no longer accessed
   Identical atoms are the **same** atom          Are totally independent of each other
   Are expensive to create and destroy            Are cheap to create and destroy
   Are indexed for fast searching                 Can't be searched except in a few cases
   An Atom has an associated table of Values      Values can only hold type-specific data
   A Link Atom may reference other Atoms          A Value can **be** a **reference** to an Atom
============================================   ============================================

Lastly, I want to mention bit about where to start looking if you have additional questions or want to get a deeper understanding of a particular topic.  I gleaned most of the information in this chapter from the built-in reference accessible through the Scheme shell.

You can run :code:`,describe` in the Scheme shell, or :code:`,d` for short, to print usage info for any command.  For example:

.. code-block:: scheme

   ,d cog-set-values!

And similarly, you can run :code:`,appropos cog`, or :code:`,a cog` for short, to get a listing of all available OpenCog functions.

Finally, the wiki reference for the OpenCog Scheme interface can be found here: `<https://wiki.opencog.org/w/Scheme>`_  I always have a browser tab open on this page.

Next Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`



