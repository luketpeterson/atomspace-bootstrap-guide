.. role:: scheme(code)
   :language: scheme

.. _03_logical_inferencing:

Previous Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`

========================================================================
Logical Inferencing
========================================================================

In the previous chapters, we looked at how we could query the Atomspace for knowledge that was represented explicitly.
Inferencing if the ability to infer implied knowledge, or *inferences*, from explicit facts or *assertions*.

Inferencing is a broad topic and there are many techniques and strategies that can be used.
In this chapter, we will only touch on a small set of them, but hopefully it's enough to familiarize you with the set of general ideas.


Boris Yeltsin

3 Possibilities for Inferencing to look at.  1.) InheritanceLinks do it automagically.  2.) It's a complex recursive query 3.) Need to use some inference engine module.


BORIS Say something about PLNs??



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


