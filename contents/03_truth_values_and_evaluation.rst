.. role:: scheme(code)
   :language: scheme

.. _03_truth_values_and_evaluation:

Previous Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`

========================================================================
TruthValues & Predicate Evaluation
========================================================================

In the previous chapter, we used a :code:`MeetLink` to query the "weight_in_kg" of "Fido the Dog",
and used a different query to find all dogs (all atoms actually) heavier than 15kg.
But how can we ask "Is Fido heavier than 15kg?".

More generally, how do we compose a conditional expression in Atomese?

.. code-block:: scheme

    (cog-evaluate!
        (GreaterThan
            (Number 10)
            (Number 2)
        )
    )

BORIS
2 ideas to make this work: 1.) Do an execute on a Get, in order to boil it down to a single token
2.) Figure out how to do a general-purpose variable grounding.  Should figure out 2, because it'll have potential implications elsewhere.

.. code-block:: scheme

    (Evaluation
        (GreaterThan
            (VariableNode "$v1")
            (Number 10)
        )
        (StateLink
            (ListLink
                (Concept "Fido the Dog")
                (Predicate "weight_in_kg")
            )
            (VariableNode "$v1")
        )
    )


.. code-block:: scheme

    (cog-evaluate!
        (StateLink
            (ListLink
                (Concept "Fido the Dog")
                (Predicate "weight_in_kg")
            )
            (VariableNode "$v1")
        )
        (GreaterThan
            (VariableNode "$v1")
            (Number 10)
        )
    )



BORIS, talk about grounding and checking if an assertion is true or not




TruthValues & EvaluationLinks
------------------------------------------------------------------------

BORIS Below is WRONG!
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