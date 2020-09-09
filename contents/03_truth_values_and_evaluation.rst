.. role:: scheme(code)
   :language: scheme

.. _03_truth_values_and_evaluation:

Previous Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`

========================================================================
TruthValues & Predicate Evaluation
========================================================================





TruthValues & EvaluationLinks
------------------------------------------------------------------------

In the previous chapter, we showed how :code:`cog-execute!` could execute certain *Active* links, resulting in an atom or value being created and returned.
For *Declarative*, aka passive links, the :code:`cog-evaluate!` OpenCog function is its counterpart.
Unlike Active Links, Declarative links always evaluate to a *TruthValue*.



BORIS, include the fact that a truthValue is attached to an atom with a special key.  Explained in values.scm example.


BORIS Let's ask the Atomspace a true/false question.  "Is Fido an Animal?"

BORIS.  Some operations result in less truth or less certainty


BORIS explain how to interpret the (stv 1 1) that is returned
BORIS What to say about EvaluationLink??  We've already introduced them above, GreaterThanLink is an EvalLink.


Explain the theory behind different kinds of truth value.




TruthValue

BORIS General overview of truth values, different types of Truth Values.

BORIS STV

BORIS Revisit PredicateNode
BORIS introduce StrengthOf & CondfidenceOf

BORIS EvaluationLink

BORIS two views, as an assertion with a truth value, or as a way to evaluate the truth of a proposition

Assert, (Come up with an example that isn't an "isa" relationship.  Dogs chew bones, goats chew leaves)

BORIS BORIS, How do I query whether something is part of another set????


BORIS introduce cog-eval!

BORIS PredicateFOrmula