.. role:: scheme(code)
   :language: scheme

.. _02_representing_knowledge:

Previous Chapter: :ref:`Getting Oriented <01_getting_oriented>`

========================================================================
Representing Knowledge
========================================================================

In the last chapter, we learned how we can think of the Atomspace as a database of documents.  Each Atom is like a document that can contain a set of key-value pairs.
But document-store database solutions are already plentiful and the value of the Atomspace comes from other capabilities.

So now we'll begin to explore the Atomspace as a knowledge representation format, or *Knowledge Base*; Often abbreviated as *KB*.

KBs hold a collection of concepts and define them in relationship to other concepts.  KBs are a relatively old idea, not unique to the Atomspace, and there is a Wikipedia page covering the basics: `<https://en.wikipedia.org/wiki/Knowledge_base>`_

This guide and nearly every other piece of documentation on KBs I've encountered describe the primitives of knowledge representation using concrete examples from the real world, such as "A dog is an animal."  This is helpful for learning the particulars of the KB because the fundamental conceptual relationships are already familiar.
However, I believe a large amount of the value of KBs and ontologies come from the ability to reason within an abstract systems of precise relationships where that precision has been imposed elsewhere, at another level, or where ambiguity never existed to begin with.  For example, a KB could be used to reason about the behavior of a computer program, given a hypothetical set of inputs.

In general, strictly regular KBs have many limitations when representing data from real world.  For example, the `InheritanceLink <https://wiki.opencog.org/w/InheritanceLink>`_ documentation points out the difference between Extensional vs Intentional Inheritance.  I'd argue this is just the tip of the iceberg when trying to formalize knowledge about the fuzzy real world into a crisp ontology.
That being said, the Atomspace is better than most other KB formats, because of the tools it provides for expressing nuance, partial truth and uncertainty.  All the same, in my opinion, a symbolic KB alone is not sufficient for many aspects of real-world knowledge representation.  Regardless, I'll attempt to refrain from injecting too much of my own personal opinions and conclusions into this guide dedicated to understanding the Atomspace.

Ultimately, any formal language is only as precise as the axioms and definitions it is built upon, and you will have to define your own grammar.  Personally I view the Atomspace less as a language and instead as the building-blocks out of which a language can be created.

PredicateNode & Links to make statements
------------------------------------------------------------------------

We touched on :code:`PredicateNode` in the last chapter when we used them as keys for values we associated with other atoms.
In grammar, a statement is divided into subject(s) and a predicate.  "The dog barks."  'Barks" is the predicate.
In the statement: "The dog is happy", "is happy" is the predicate.

The important thing about predicates is that they allow assertions to be made.  Concepts by themselves don't say anything, they merely exist, but predicates allow for statements.

In the Atomspace, a :code:`PredicateNode` provides a label for these predicate concepts.
A more formal description of a :code:`PredicateNode` in the Atomspace is here: `<https://wiki.opencog.org/w/PredicateNode>`_

As seen in last chapter, the below Scheme snippet tells the Atomspace that "Fido the Dog's weight is 12.5kg".

.. code-block:: scheme

   (cog-set-value! (Concept "Fido the Dog")
      (Predicate "weight_in_kg") (FloatValue 12.5))

Now, I can retrieve the :code:`weight_in_kg` using the :code:`cog-value` command, as demonstrated previously, but I cannot search for all dogs whose weight is under 15kg.
Values associated with atoms are not searchable.  If I have the atom, I can get a value attached to it, but if I am looking for atoms with an associated value that meets some criteria, this approach won't work.

Here is another way to express the statement "Fido the Dog's weight in kg is 12.5".

.. code-block:: scheme

   (StateLink
      (ListLink
         (Concept "Fido the Dog")
         (Predicate "weight_in_kg")
      )
      (NumberNode 12.5)
   )

Let's break down exactly what we just did.  We created two new link atoms and a new :code:`NumberNode` node atom.

.. image:: images/fidos_weight.svg
   :height: 300px
   :width: 450 px
   :scale: 100 %
   :alt: Fido's Weight Graphic
   :align: center

Here is a graphical representation of the atom relationships we just expressed.

In the inner-part of the expression, we created a :code:`ListLink` atom that references :code:`(Concept "Fido the Dog")` and :code:`(Predicate "weight_in_kg")`.
This :code:`ListLink` is just a simple association between the other two node atoms.

The formal description of :code:`ListLink` tells us we should think of this as an argument list, and from a programming language perspective this makes sense.
Personally, however, I prefer to think about it from a natural language perspective.
By definition, this particular :code:`ListLink` atom is *the* :code:`ListLink` atom that references :code:`(Concept "Fido the Dog")` and :code:`(Predicate "weight_in_kg")` in that order.
Therefore, in this context, you can think of it as the atom that means "Fido the Dog's weight in kg".
Basically, a single atom, i.e. our new link atom, is able to represent a compound concept created by combining two other atoms.

The documentation for :code:`ListLink` is here: `<https://wiki.opencog.org/w/ListLink>`_, if you want to understand it more precisely.  

Moving on, the outer part of the expression creates a :code:`StateLink`.  The :code:`StateLink` atom that we just made references our newly-created :code:`ListLink` and a newly-created :code:`NumberNode` that has the "label" of "12.5".
A :code:`StateLink` is like a :code:`ListLink` insofar as it also references other atoms and provides a way to reference this newly combined concept as an atom itself.

The main feature of a :code:`StateLink` is that there can be only one :code:`StateLink` for each referant in position 0 of the :code:`StateLink`'s outbound set.
So, referring back to our example, "Fido the Dog's weight in kg" can only have one :code:`StateLink` associated with it.
In plain English, "Fido the Dog's weight in kg" can only be one thing at a time.  His weight can't simultaneously be 12.5kg and 15kg.  Setting it to 15kg will update the :code:`StateLink` atom that's already there, rather than creating a new atom.

The documentation for :code:`StateLink` is here: `<https://wiki.opencog.org/w/StateLink>`_.

In addition, more documentation and examples along these lines can be found in these OpenCog examples: `<https://github.com/opencog/atomspace/blob/master/examples/atomspace/state.scm>`_ & `<https://github.com/opencog/atomspace/blob/master/examples/atomspace/property.scm>`_

A Simple Query using GetLink & VariableNode
------------------------------------------------------------------------

So, I've told the Atomspace that "Fido the Dog's weight in kg is 12.5".  How can I retrieve that information?  How do I ask "What is Fido the Dog's weight in kg?"

Like this:

.. code-block:: scheme

   (cog-execute!
      (GetLink
         (StateLink
            (ListLink
               (Concept "Fido the Dog")
               (Predicate "weight_in_kg")
            )
            (VariableNode "$v1")
         )
      )
   )

We'll go through what we just did, step by step.  But first, I want to rewrite the above statement so our code can be a little less verbose and we can focus on what really matters.

.. code-block:: scheme

   (define fidos_weight_link (List
      (Concept "Fido the Dog")
      (Predicate "weight_in_kg")))

Since Fido's weight is something we're referencing often, we can use Scheme's :scheme:`define` feature to create a single token to refer to it.

Now our query looks like this:

.. code-block:: scheme

   (cog-execute!
      (Get
         (State
            fidos_weight_link
            (Variable "$v1")
         )
      )
   )


Just like we abbreviated :code:`ConceptNode` and :code:`PredicateNode` earlier, we can abbreviate :code:`ListLink` as just :code:`List` and :code:`StateLink` as :code:`State`.
Now that I've introduced them, I'll also start abbreviating :code:`GetLink` as :code:`Get`, :code:`VariableNode` as :code:`Variable`, etc.  You get the idea, so I won't explicitly explain abbreviations from here onward.

Anyway, let's get to the meat of what we just did.  First, notice the :code:`cog-execute!` function call.
This is invoking an OpenCog function which tells the Atomspace to execute a link.  What does it mean to execute a link?  

So far, the links we've seen, like the :code:`ListLink` and :code:`StateLink` we used above have just been declarative.
But other types of links are "executable", meaning they perform a function and can alter the Atomspace.
Executing a link can create new atoms in the Atomspace or even delete existing atoms.

:code:`GetLink` is one of the executable link types.  Executing a :code:`GetLink` performs a query in the Atomspace, and returns the atoms found by the query.

Let's look at the atom that our :code:`GetLink` is referencing.  This atom is our query:

.. code-block:: scheme

   (State
      fidos_weight_link
      (Variable "$v1")
   )

This can be thought of as a "Match Expression", because executing the :code:`GetLink` will search the Atomspace for all atoms that match this atom we provided.
The :code:`VariableNode` can then be thought of as the wildcard.  The wildcard can match any other atom.
If you are familiar with `Regular Expressions <https://www.regular-expressions.info/quickstart.html>`_, this is the same principle.

So, you might interpret this query expression as saying "Find all the :code:`StateLink` atoms that connect :code:`fidos_weight` to *something*.
What are all the *somethings* that you found?"

When we execute our query, it should return:

.. code-block:: scheme

   (SetLink
      (NumberNode "12.5"))

You probably spotted our :code:`(NumberNode "12.5")` atom.  That was matched by the :code:`VariableNode` in the query atom, but what's with the :code:`SetLink`?

:code:`cog-execute!` returns a :code:`SetLink` to us, which is similar to a :code:`ListLink` except that :code:`SetLink` is unordered.
:code:`cog-execute!` returns a :code:`SetLink` instead of a "naked" node atom because a query may match more than one atom and there is no way to know the number of results that will be found, in the general case.

Lastly, let's get our query result back into Scheme.  The Scheme snippet below adds 50 to Fido's weight, just like the example from the previous chapter.

.. code-block:: scheme

   (+
      (cog-number
         (car
            (cog-outgoing-set
               (cog-execute!
                  (GetLink
                     (State
                        fidos_weight_link
                        (VariableNode "$v1")
      )  )  )  )  )  )
      50
   )

Because :code:`cog-execute!` returns a :code:`SetLink` to us, we must get the first element of the :code:`SetLink`, which will be a :code:`NumberNode`.  We can extract our numerical value from that :code:`NumberNode`.
We use the :code:`cog-outgoing-set` OpenCog function to convert the :code:`SetLink` into a Scheme list, and then use :scheme:`car` to extract the first element of that list.

Finally, we can use the :code:`cog-number` OpenCog function to convert the :code:`NumberNode` into a Scheme number, before performing the arithmetic in Scheme.

.. note:: Much of the documentation is written to feature :code:`GetLink` instead of :code:`MeetLink`, and :code:`BindLink` instead of :code:`QueryLink`.  The only semantic difference is that :code:`MeetLink` and :code:`QueryLink` return results as a :code:`QueueValue` which is transient, while :code:`GetLink` and :code:`BindLink` return a :code:`SetLink` which will become part of the Atomspace until it is deleted.

.. note:: QUESTION for someone smarter than me. Why does (cog-value-ref) give me "index out of range" errors on QueueValues??  It seems like this should be something that works.

That's probably enough on this simple query.  If you want a more complete explanation, the documentation for :code:`VariableNode` is here: `<https://wiki.opencog.org/w/VariableNode>`_ and the documentation for :code:`GetLink` is here: `<https://wiki.opencog.org/w/GetLink>`_

More Elaborate Queries with Other Link Types
------------------------------------------------------------------------

This is a good place to introduce the concepts of *Grounded* vs *Ungrounded* expressions.
The formal definition is that ungrounded expressions contain 1 or more *Free* :code:`VariableNode` atoms, while grounded expressions don't contain any.
Personally, the way I think about it is that grounded expressions are statements and ungrounded expressions are questions.

Just as in English, questions and statements can take a similar gramatical form.  Consider this example. 
Statement: "The man is running."  Question: "Who is running?" Answer: "The man".

The question-word "Who" in this example is like a :code:`VariableNode`.
When the question is matched against the statement, the relative gramatical position of the word "Who" indicates which part of the statement will appropriately answer the question.

So, another intuition for :code:`GetLink` is that it takes an ungrounded expression and returns a grounded expression.
Or said another way, it takes a question and returns an answer.

So let's flip our previous question inside out.  Consider this query:

.. code-block:: scheme

   (cog-execute!
      (Get
         (State
            (Variable "$v1")
            (Number 12.5)
         )
      )
   )

Our previous question was: "What is Fido the Dog's weight in kg?".  Now our question is: "What value is 12.5?".
Executing that snippet should return our :code:`ListLink` that represents Fido's weight.

You've probably noticed the :code:`VariableNode` atom's label, :code:`"$v1"`.  This is just an arbitrary label, no different than the other labels we've used such as "Fido the Dog".
The Atomspace allows you to use multiple :code:`VariableNode` atoms to compose compound questions.
For example the English question: "What cities in Germany are on the river Danube?" is a compound question because it has two parts, "In Germany" and "On the river Danube".

Soon we'll get to the uses for multiple :code:`Variable` nodes within a query.
Right now, let's give Fido a friend by executing this Scheme snippet:

.. code-block:: scheme

   (StateLink
      (ListLink
         (Concept "Fluffy the Dog")
         (Predicate "weight_in_kg")
      )
      (NumberNode 17)
   )

Now, I want to ask the Atomspace to find the dog that has a weight over 15kg.  My query looks like this:

.. code-block:: scheme

   (cog-execute!
      (QueryLink
         (And
            (State
               (List
                  (Variable "dog_node")
                  (Predicate "weight_in_kg")
               )
               (Variable "dogs_weight_node")
            )
            (GreaterThan
               (Variable "dogs_weight_node")
               (Number 15)
            )
         )
         (Variable "dog_node")
      )
   )

We found Fluffy!

Now, let's go over the new Link types I just introduced, and I'll explain the query along the way.

BindLink
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:code:`BindLink` is another way to execute a query.  It is just like the :code:`GetLink` atom, that we used in the previous examples, except that :code:`BindLink` allows us to declare the format for the query results.

To understand this better, try this nearly identical version of the query using :code:`GetLink` instead of :code:`BindLink`.

.. code-block:: scheme

   (cog-execute!
      (Get
         (And
            (State
               (List (Variable "dog_node") (Predicate "weight_in_kg"))
               (Variable "dogs_weight_node"))
            (GreaterThan (Variable "dogs_weight_node") (Number 15))
   )  )  )

As you can see, it also returns the "Fluffy the Dog".  But unlike the :code:`BindLink` version, the returned atom is a bit more cluttered.

The :code:`GetLink` version returns:

.. code-block:: scheme

   (SetLink
      (ListLink
         (ConceptNode "Fluffy the Dog")
         (NumberNode "17")))

While the :code:`BindLink` version returns:

.. code-block:: scheme

   (SetLink
      (ConceptNode "Fluffy the Dog"))

That is because we explicitly told the :code:`BindLink` atom that we were interested in :code:`(Variable "dog_node")` as our result.  On the other hand, the :code:`GetLink` atom created a :code:`ListLink` referencing all of the :code:`VariableNode` atoms in our query.

BORIS NOTE: MeetLink is GetLink and QueryLink is BindLink

You can think of :code:`BindLink` as performing two operations in sequence.  First, it performs a query to search for matching atoms, and then it performs a subsequent step to format the results as new atoms.

In fact, the "get-put.scm" OpenCog example demonstrates exactly how a :code:`BindLink` is actually composed from a :code:`GetLink` and a :code:`PutLink`.  I recommend going through that example as well as the "bindlink.scm" example, which can be found here: `<https://github.com/opencog/atomspace/blob/master/examples/atomspace/bindlink.scm>`_ & `<https://github.com/opencog/atomspace/blob/master/examples/atomspace/get-put.scm>`_

AndLink
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:code:`AndLink` is a link atom type for performing the binary "And" operation.  You probably guessed that from its name.
So, for a query to match, both sides of the :code:`AndLink` must be satisfied.

Back to our example:

.. code-block:: scheme

   (And
      (State
         (List
            (Variable "dog_node")
            (Predicate "weight_in_kg")
         )
         (Variable "dogs_weight_node")
      )
      (GreaterThan
         (Variable "dogs_weight_node")
         (Number 15)
      )
   )

This query's use of :code:`And` is essentially saying "Find an atom connected to the *weight_in_kg* atom with a :code:`ListLink` that itself is connected to another atom by a :code:`StateLink` **AND** the numerical value of that other atom is greater than 15."

Try experimenting a bit with this query.  For example, if you change the query to compare against :code:`(Number 10)` instead of :code:`(Number 15)`, you will find the query returns both Fido and Fluffy.

Moving on, notice that the :code:`(Variable "dogs_weight_node")` atom occurs in both side of the :code:`And` expression.  This is important.  
One way to think about this is that the :code:`(Variable)` node is "defined" or temporarily given a value by the first side of the :code:`And` expression, and then that value is used when evaluating the second side.

As somebody with a strong background in procedural programming, this conceptualization resonates with me.  However, if your intuition comes from databases, you may want to think of the operation as an "INNER JOIN" from SQL.  These mental models are functionally equivalent.

If you're curious, the Atomspace has an 

GreaterThanLink
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

BORIS

BORIS, split the query section in two.  Create a query that returns just Fido's concept node, then fluffy and fido together.
Query in such a way that only Fido is found
Query to find both dogs
BORIS




BORIS HERE IS WHERE WE ADD GREATER THAN Links
ALSO THROW IN An AND for good measure


I recommend exploring queries and executable links further by going through the "assert-retract.scm" OpenCog example here: `<https://github.com/opencog/atomspace/blob/master/examples/atomspace/assert-retract.scm>`_






TruthValues & EvaluationLinks
------------------------------------------------------------------------

In the previous section, we showed how :code:`cog-execute!` could execute certain types of links, resulting in an atom being returned.  Now we'll look at the :code:`cog-evaluate!` OpenCog function.

Now that we know how to ask the Atomspace about Fido's weight, the next logical step is to see if we can ask it a yes/no (true/false) question.  "Is Fido the Dog's weight greater than 10kg?"  I phrase that question like this:

.. code-block:: scheme

   (define fidos_weight_query
      (GetLink
         (State
            fidos_weight_link
            (VariableNode "$v1")
         )
      )
   )

   (cog-evaluate! 
      (GreaterThanLink
         fidos_weight_query
         (NumberNode 10.0)
      )
   )


BORIS explain how to interpret the (stv 1 1) that is returned
BORIS What to say about EvaluationLink??  We've already introduced them above, GreaterThanLink is an EvalLink.







BORIS MEMBERLINK!!!!

A Few Simple Links - InheritanceLink, MemberLink, ListLink
------------------------------------------------------------------------

Every KB that I'm aware of has two special link types for specialization and generalization.  Atomspace is no different.
They are called *InheritanceLink* and *SubsetLink*

InheritanceLink is the specialization link.  It is essentially equivalent to the "Is a" statement.  e.g. a dog is an animal.
The name "Inheritance" is appropriate even if it feels a little odd at first. Consider making the dog statement in a more verbose form; you might say "All statements about an animal are also true about a dog.", or alternatively, "A dog inherits the relationships of an animal."
InheritanceLink is described in more detail here: `<https://wiki.opencog.org/w/InheritanceLink>`_

Here is an example Scheme snippet to create an InheritanceLink.

.. code-block:: scheme

   (Inheritance
      (Concept "Dog")
      (Concept "Animal"))

Executing the above Scheme code creates or references the ConceptNodes, "Dog" and "Animal", and then creates an InheritanceLink between them.  So the Atomspace will have at least 3 atoms after executing the above code, two ConceptNodes and one InheritanceLink.

.. note:: "Inheritance" is an abbreviated alias of "InheritanceLink", and "Concept" is an alias of "ConceptNode".  They are functionally identical.

SubsetLink is the generalization link, making it the inverse of the InheritanceLink.  For example, you could say "one type of animal is a dog."  More documentation on SubsetLink is here: `<https://wiki.opencog.org/w/SubsetLink>`_

Here is an example Scheme snippet to create a SubsetLink.

.. code-block:: scheme

   (SubsetLink
      (ConceptNode "Dog")
      (ConceptNode "Animal"))

While they may make complimentary assertions, the referant order for SubsetLink and InheritanceLink is the same.  This trips me up sometimes.

.. note:: QUESTION for someone smarter than me: Why doesn't a SubsetLink imply an InheritanceLink?  What's the point of two separate link types at all? It seems that a more regularized structure would have the same link appear as both depending on the query. 


BORIS, Ordered list link, colors of the rainbow


Logical Inference
------------------------------------------------------------------------

BORIS

Let's start out by assigning a property to some concepts.  In this case, let's define the number of wheels each object has with this Scheme snippet.



BORIS HOW to EXPRESS SOME statements, e.g. how many wheels does something have.  Bicycle: 2. Car 4. Animal 0.


BORIS HOW TO query the number of wheels.

BORIS EvaluationLink???

A dog has 0 wheels.  Dog implies Animal, Animal has 0 wheels.



Querying the Atomspace
------------------------------------------------------------------------

BORIS Answering a simple question


EvaluationLink & Truth Values
------------------------------------------------------------------------

BORIS BORIS, How do I query whether something is part of another set????









Defining new Types
------------------------------------------------------------------------
Building up our own grammar.
BORIS Defining some 

A DefineLink??? https://wiki.opencog.org/w/DefineLink

It is advised to use an EquivalenceLink instead of a DefineLink
https://wiki.opencog.org/w/EquivalenceLink




Is TypedAtomLink the way???  https://wiki.opencog.org/w/TypedAtomLink
Or SignatureLink??  https://wiki.opencog.org/w/SignatureLink



Truth Values and Values in General
------------------------------------------------------------------------
BORIS.  Some operations result in less truth or less certainty
