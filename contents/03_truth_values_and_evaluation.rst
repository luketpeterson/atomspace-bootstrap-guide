.. role:: scheme(code)
   :language: scheme

.. _03_truth_values_and_evaluation:

Previous Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`

========================================================================
TruthValues & Predicate Evaluation
========================================================================

In the previous chapter, we used a :code:`MeetLink` and :code:`QueryLink` to query the "weight_in_kg" of "Fido the Dog",
and used a different query to find all dogs (actually all atoms) heavier than 10kg.
But how can we ask "Is Fido heavier than 10kg?".

The Philosophy of Truth
------------------------------------------------------------------------

BORIS.  not sure whether it makes sense to first explain complex truth, or the Atomese equivalent of branching.

More generally, how do we compose a conditional (Boolean) expression in Atomese?

In a simple form, like this:

.. code-block:: scheme

    (cog-evaluate!
        (GreaterThan
            (Number 10)
            (Number 2)
        )
    )

Notice that we've traded :code:`cog-execute!` for :code:`cog-evaluate!`.
These OpenCog functions are similar, but where :code:`cog-execute!` may return anything at all, :code:`cog-evaluate!` will always return a *TruthValue*.

Anyway, when you ran that :code:`cog-evaluate!` snippet above, you should have gotten this:

.. code-block:: scheme

    (stv 1 1)

"stv" in this case stands for *Simple Truth Value*, and an STV is composed of two floating point numbers: *Strength* and *Confidence*.
In our case, they are both exactly 1.  The expression was 100% true, and we are 100% sure of that.

So, as you can see, this is a step beyond simple bivalent (crisp true or false) logic in both reasoning ability and complexity.

But what precisely does it mean for something to be half-true?  Well... It's complicated.

Consider the statement "Charlie is tall."  If Charlie were 210cm tall, most people today would judge that true.
If he were 120cm, most would judge it false.  But what if Charlie were 175cm?  In this case, the statement might be "half-true".

This line of reasoning was formalized as `Fuzzy Logic <https://en.wikipedia.org/wiki/Fuzzy_logic>`_, by Lotfi Zadeh, whom I was lucky enough to chat with for half an hour, mostly about self-driving cars, back in the year 2000 when I was 19 years old, but I digress...
Using fuzzy logic, we could create a set for all tall people, and then a person with a height of 175cm could have a 50% membership in that set.

This is conceptually different from the statement "The train from Manchester arrives every day at 10:42am."  Given the legendary unreliability of the London Midlands train service, you'd also assign that statement a low truth value.
But it is a probabalistic truth rather than a partial or fuzzy truth.  Some days, the train will indeed arrive on time, but on the majority of days it will not.  This kind of truth value is meant to express a probability that the statement is true.

BORIS.  A lot MORE to say,  PDFs, and also beysian models. but don't get too deep into theory.

Conditional Expressions
------------------------------------------------------------------------



.. code-block:: scheme

    (cog-evaluate!
        (SatisfactionLink
            (AndLink
                (StateLink
                    (ListLink
                        (Concept "Fido the Dog")
                        (Predicate "weight_in_kg")
                    )
                    (VariableNode "dogs_weight_node")
                )
                (GreaterThan
                    (VariableNode "dogs_weight_node")
                    (Number 10)
                )
            )
        )
    )








Declaring EvaluationLinks
------------------------------------------------------------------------

BORIS, talk about grounding and checking if an assertion is true or not

Assert, (Come up with an example that isn't an "isa" relationship.  Dogs chew bones, goats chew leaves)

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


BORIS.  Explain AnchorNodes and VariableLists




BORIS Revisit PredicateNode
BORIS introduce StrengthOf & CondfidenceOf

BORIS EvaluationLink

BORIS two views, as an assertion with a truth value, or as a way to evaluate the truth of a proposition


BORIS BORIS, How do I query whether something is part of another set


BORIS PredicateFOrmula



BORIS Cover using PutLink to find a location and update it.  For example, search the Atomspace, and put all dogs heavier than 10kg is the "Big Dogs" set.
