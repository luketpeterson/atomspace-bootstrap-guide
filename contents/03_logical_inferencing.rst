.. role:: scheme(code)
   :language: scheme

.. _03_logical_inferencing:

Previous Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`

========================================================================
Logical Inferencing
========================================================================


Boris Yeltsin


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


BORIS.  Some operations result in less truth or less certainty


BORIS explain how to interpret the (stv 1 1) that is returned
BORIS What to say about EvaluationLink??  We've already introduced them above, GreaterThanLink is an EvalLink.


Explain the theory behind different kinds of truth value.




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

Check out this guide:
https://wiki.opencog.org/w/Adding_New_Atom_Types

A DefineLink??? https://wiki.opencog.org/w/DefineLink

It is advised to use an EquivalenceLink instead of a DefineLink
https://wiki.opencog.org/w/EquivalenceLink




Is TypedAtomLink the way???  https://wiki.opencog.org/w/TypedAtomLink
Or SignatureLink??  https://wiki.opencog.org/w/SignatureLink


