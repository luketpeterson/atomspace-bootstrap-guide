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

More generally, how do we compose a Boolean expression in Atomese?

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

The Philosophy of Truth
------------------------------------------------------------------------

When you run that :code:`cog-evaluate!` snippet above, you should get this:

.. code-block:: scheme

    (stv 1 1)

"stv" in this case stands for *Simple Truth Value*, and an STV is composed of two floating point numbers: *Strength* and *Confidence*.
In our case, they are both exactly 1.  The expression was 100% true, and we are 100% sure of that.

So, as you can see, this is a step beyond simple bivalent (crisp true or false) logic in both reasoning ability and complexity.

But what precisely does it mean for something to be half-true?  Well... It's complicated.

Consider the statement "Charlie is tall."  If Charlie were 210cm tall, most people today would judge that true.
If he were 120cm, most would judge it false.  But what if Charlie were 175cm?  In this case, the statement might be "half-true".

This line of reasoning was formalized as `Fuzzy Logic <https://en.wikipedia.org/wiki/Fuzzy_logic>`_, by Lotfi Zadeh, whom I was lucky enough to chat with for half an hour, mostly about self-driving cars, back in the year 2000 when I was 19 years old, but I digress...

Using fuzzy logic, we can create a set for all tall people, and then a person with a height of 175cm could have a 50% membership in that set.
In traditional set theory, a data point either belongs or doesn't belong in a set, based on the set membership function.  The set always has a crisp boundary.  In fuzzy logic, the membership function returns a value between 0 and 1, so there can be a continuous transition from outside the set to inside the set.

But consider the conceptual difference between our statement about Charlie and the statement "The train from Birmingham arrives every day at 10:42am."  Given the legendary unreliability of the London Midland train service, you'd also assign that statement a low truth value.
But this is a probabilistic truth rather than a fuzzy truth.  Some days, the train will indeed arrive on time, but on the majority of days it will not.  This kind of truth value is meant to express a probability that the statement is true.

So in summary, a fuzzy truth value represents the **degree** to which a statement is true, while a probabilistic truth value represents the **chance** that it is true.
Fuzzy truth values are useful for tracking, well fuzzy, statements of known facts, while probabilistic truth values are useful for tracking predictions and known uncertainties.
They are related concepts, but they aren't mathmatically interchangeable.

Those are two interpretations of the *strength* component; what about the the *confidence* component?
Strength represents the known aspect of the truth value and confidence is the unknown aspect.
Consider a truth value of :scheme:`(stv 0.5 1.0)` for the statement "A coin-flip will land on heads."  If somebody offered you a bet with better-than-even odds on that coin, you could be confident that your expected return would be positive.
But consider the same statement about an unknown coin :scheme:`(stv 0.5 0.0)`.  It might be a weighted coin that lands on tails 99% of the time.  From that TruthValue you just don't know.

OpenCog and the Atomspace support additional types of more complicated TruthValues to cover different situations.
For example there is the `FormulaTruthValue <https://wiki.opencog.org/w/FormulaTruthValue>`_ for situations where the truth of an assertion depends on additional factors.  These are good for representing probability distribution functions.
Also there is the `CountTruthValue <https://wiki.opencog.org/w/TruthValue#CountTruthValue>`_ for situations where the system continues to collect new observations and refine its assesment of the probability.

Partial truth is a very big topic, and we're not going to be able to do it justice in this guide.  This section is just a superficial introduction to make you aware of the problem-space.

In general, you can read the official OpenCog reference for TruthValue here: `<https://wiki.opencog.org/w/TruthValue>`_

And now we'll introduce *Probabilistic Logic Networks*, or *PLNs* for short.  PLNs are a way to reason with partial truth values.
OpenCog and PLNs have a shared heritage, and many ideas from PLNs deeply inform the architecture of OpenCog.  We'll talk a lot more about PLNs in the coming chapters.

For now, you can read an introductory paper on PLNs here: `<https://aiatadams.files.wordpress.com/2016/02/invited_paper_3.pdf>`_

And the complete PLN book can be downloaded (for now) here: `<https://aiatadams.files.wordpress.com/2016/02/pln_book_6_27_08.pdf>`_

Conditional Expressions
------------------------------------------------------------------------

So we saw above how we could use :code:`cog-evaluate!` to evaluate a atom to generate a TruthValue.
But how do we utilize that result to control what our program does next?
In other words, what are the Atomese equivalents for program-flow constructs like If-Then statements, Case statements, etc.?

If you are as steeped in procedural programming as I am, this next bit will require you to turn your brain inside-out.
If you come from a `Lambda Calculus <https://en.wikipedia.org/wiki/Lambda_calculus>`_ or `Functional Programming <https://en.wikipedia.org/wiki/Functional_programming>`_ background then, lucky you!
This next part will be a lot easier to wrap your head around.

Let's bring back Fido.

.. code-block:: scheme

    (define fidos_weight_link
        (ListLink
            (Concept "Fido the Dog")
            (Predicate "weight_in_kg")
        )
    )

    (StateLink
        fidos_weight_link
        (NumberNode 12.5)
    )

Now, we want to put him into a "Big Dog" or a "Small Dog" set, depending on his weight.
But first, we need to define a predicate that will evaluate to true if his weight is above a threshold.

.. code-block:: scheme

    (define fido_is_big?
        (SatisfactionLink
            (AndLink
                (StateLink
                    fidos_weight_link
                    (VariableNode "dogs_weight_node")
                )
                (GreaterThan
                    (VariableNode "dogs_weight_node")
                    (Number 15)
                )
            )
        )
    )

Let's stop here, and just evaluate our new predicate.

.. code-block:: scheme

    (cog-evaluate! fido_is_big?)

You should get back :scheme:`(stv 0 1)`, aka false.  Fido is not heavier than 15kg.  If you're not convinced, try tweaking Fido's weight or the predicate to get the answer you want.

Earlier I promised I wouldn't drop a new atom or other construct on you without at least attempting to demystify it, so :code:`SatisfactionLink` is yet another query link type.
Fundamentally it's just like :code:`MeetLink`, :code:`GetLink`, :code:`QueryLink`, and :code:`BindLink`.
The main feature that sets :code:`SatisfactionLink` apart is that it evaluates to a TruthValue.  True, aka :scheme:`stv(1, 1)`, if the expression could be matched in the Atomspace, and false, aka :scheme:`stv(0, 1)`, if not.

So unlike the other query link types, :code:`SatisfactionLink` is appropriate to use in an evaluation context rather than in an execution context.  In fact, :code:`SatisfactionLink` is actually the basic building-block from which all of those other query links are constructed.

Continuing on, we can now create the appropriate :code:`MemberLink`, depending on how our predicate evaluates.

.. code-block:: scheme

    (cog-evaluate!
        (OrLink
            (AndLink
                fido_is_big?
                (MemberLink
                    (Concept "Fido the Dog")
                    (Predicate "Big Dog")
                )
            )
            (MemberLink
                (Concept "Fido the Dog")
                (Predicate "Small Dog")
            )
        )
    )
    

BORIS this is BORKED.  The trouble is that those memberlinks end up existing in the atomspace BECAUSE they exist as part of the query!!!

.. code-block:: scheme

    (cog-evaluate!
        (OrLink
            (AndLink
                fido_is_big?
                (StateLink
                    (Concept "Fido the Dog")
                    (Predicate "Big Dog")
                )
            )
            (StateLink
                (Concept "Fido the Dog")
                (Predicate "Small Dog")
            )
        )
    )


    (cog-evaluate!
        (MemberLink (stv 1 1)
            (Concept "Fido the Dog")
            (Predicate "Small Dog")
        )
    )




BORIS, talk about how both sides can potentially execute, and it's just up to the end to decide which side to use.  How there isn't a program counter, as in precedural programming.



Boris, what happens if something has a truth value of 0.5???  Which link is created???  Both.


BORIS YELTSIN
Talk about side-effect-free vs. side-effects, SequentialAndLink



BORIS introduce StrengthOf & CondfidenceOf



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


BORIS What to say about EvaluationLink??  We've already introduced them above, GreaterThanLink is an EvalLink.



BORIS.  Explain AnchorNodes and VariableLists




BORIS Revisit PredicateNode

BORIS EvaluationLink

BORIS two views, as an assertion with a truth value, or as a way to evaluate the truth of a proposition


BORIS BORIS, How do I query whether something is part of another set


BORIS PredicateFOrmula



BORIS Cover using PutLink to find a location and update it.  For example, search the Atomspace, and put all dogs heavier than 10kg is the "Big Dogs" set.


BORIS VariableList, Typed Variables (CAN I DEFINE MY OWN TYPES???)
BORIS Next Chapter, program segmentation, DefineLinks, Tail Recursion, etc. look at the recursive-loop.scm example.
We'll also talk about the FFI, like using ExecutionOutput and GroundedSchema, or GroundedPredicate, look at "execute.scm"