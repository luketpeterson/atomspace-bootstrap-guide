.. role:: scheme(code)
   :language: scheme
.. role:: c(code)
   :language: c

.. _04_evaluation_and_truth_values:

Previous Chapter: :ref:`Programming with Atomese <03_atomese_programming>`

========================================================================
Evaluation and Truth Values
========================================================================


























BORIS, Section below should be part of the lead-in to using SequentialAndLink and SequentialOrLink

You may have already stumbled into this, but you can use a :code:`ListLink` to execute multiple operations.
Here's an example: 

.. code-block:: scheme

    (DefineLink
        (DefinedSchemaNode "make_nighttime")
        (ListLink
            (PutLink
                (State
                    (Variable "switch_placeholder")
                    (Concept "On")
                )
                (Concept "Moonlight")
            )
            (PutLink
                (State
                    (Variable "switch_placeholder")
                    (Concept "Off")
                )
                (Concept "Sunlight")
            )
        )
    )



Local Variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

BORIS, I can't actually figure out how to use local variables in Lambdas, but I'm also not convinced they are needed.

Distilling the benefits of functions down to just argument passing really doesn't do justice to the concept.
Functions provide a means manage the side-effects that a subroutine can produce.
In other words, if a function produces 


BORIS.  Create a return value from a function.
Then create an increment function where the value passes through a local variable, think up a way that the function would cross-talk with itself from one calling to the next.
BORIS BEST IDEA, Make a recursive function that have 2-dimensions so it interferes with itself.
Consider implementing a "list_of_n_primes" function.  That might be the best way to contrive a variable confict, and if not it'll still be good practice.
Read the parallelism OpenCog Example.




BORIS CHAPTER FLOW IDEAS.
In the next chapter, introduce EvaluationLink and DefinedPredicate.
Then go on to cover the use of SequentialAnd, SequentialOr, and other constructs to compose programs.


.. code-block:: scheme

    (cog-evaluate!
    (Evaluation
        (DefinedPredicate "is_pos_integer?")
        (Number 2)
        )
    )


    (Define
        (DefinedPredicateNode "is_pos_integer?")
        ; Determines whether "x" is a positive integer, i.e. ?(x > 0 && x % 1 == 0)
        ; The lack of a native % (mod) fn turns a constant-time op into an order n op. :-(
        ; Also not numerically stable for high values of x, due to floating point rounding

        (Lambda
            (Variable "x")
            (SequentialAndLink

                ; As long as x is greater-than-or-equal-to 1, we can continue
                ; Otherwise we will return false
                (NotLink (GreaterThanLink (Number 1) (Variable "x") ) )

                (SequentialOrLink

                    ; See if our number is exactly 1, return true if so
                    (EqualLink (Variable "x") (Number 1))

                    ; Recurse with 1 minus our number
                    (Evaluation
                        (DefinedPredicateNode "is_pos_integer?")
                        (MinusLink
                            (Variable "x")
                            (Number 1)
                        )
                    )
                )
            )
        )
    )


    BORIS, looks like I will need to explain DefinedPredicate, which means I'll probably need to explain evaluation
    BORIS, write up SequentialAnd & SequentialOr, and how they fit in, 
        This will require drawing a diagram of AND, OR, and NOT gates.
    BORIS, look at PredicateFormula, it Constructs a TruthValue from two number values

.. code-block:: scheme

    (Define
        (DefinedPredicateNode "is_prime_helper")
        ; Determines whether "x" is evenly divisible by "i" or another integer greater than "i"
        ; In otherwords, returns partial NOT prime.  Intended to be called by "is_prime?"
        ; If called with i=2, false = x is prime, true = x is not prime

        (Lambda
            (VariableList
                (Variable "x")
                (Variable "i")
            )
            (SequentialAndLink

                ; If i is greater-than-or-equal-to x, return false because we've tried all possibilities, so it must be prime
                ; Ideally we could stop at sqrt(x), but if I cared about efficiency, I'd implement native modulo first
                (GreaterThan (Variable "x") (Variable "i") ) ; greater-than-or-equal is the same as not-less-than

                (SequentialOrLink
                    ; Check to see if x is evenly divisible by i, if so, return true
                    (Evaluation
                        (DefinedPredicateNode "is_pos_integer?")
                        (DivideLink (Variable "x") (Variable "i"))
                    )

                    ; Recurse with i++       
                    (Evaluation
                        (DefinedPredicateNode "is_prime_helper")
                        (Variable "x")
                        (PlusLink (Variable "i") (Number 1))
                    )
                )
            )
        )
    )

    (cog-evaluate!
    (Evaluation
        (DefinedPredicate "is_prime_helper")
        (Number 5)
        (Number 2)
        )
    )

    (Define
        (DefinedPredicateNode "is_prime?")
        ; Determines whether a number supplied is prime or not
        
        (Lambda
            (Variable "x")

            ; Call our recursive helper function
            (NotLink
                (Evaluation
                    (DefinedPredicateNode "is_prime_helper")
                    (Variable "x")
                    (Number 2)
                )
            )
        )
    )

    (cog-evaluate!
    (Evaluation
        (DefinedPredicate "is_prime?")
        (Number 37)
        )
    )

    

    BORIS, Include discussion about FFI, like a printf debug funcrtion

.. code-block:: scheme

    (define (scm-display-wrapper-exec atom)
        (display atom)
        (Concept "done")
    )

    (cog-execute!
        (ExecutionOutput
            (GroundedSchema "scm: scm-display-wrapper-exec")
            (Concept "Hi")
        )
    )

    (define (scm-display-wrapper-eval atom)
        (display atom)
        (stv 1 1)
    )

    (cog-evaluate!
        (Evaluation
            (GroundedPredicate "scm: scm-display-wrapper-eval")
            (Concept "Hi")
        )
    )

    (define (scm-display-wrapper-eval-2-arg atom1 atom2)
        (display atom1)
        (display atom2)
        (stv 1 1)
    )

    (cog-evaluate!
        (Evaluation
            (GroundedPredicate "scm: scm-display-wrapper-eval-2-arg")
            (List
                (Concept "One")
                (Concept "Two")
            )
        )
    )


Boris end of FFI section

.. code-block:: scheme


    (Define
        ; Calculates the next prime number, greater than the number supplied
        (DefinedSchemaNode "next_prime")
        (Lambda
            (Variable "x")

            ; temp = x+1
            (SetValue (Variable "x") (Predicate ""))
            
            ; Check to see if temp is prime

            ; If it is, return it, if not, recurse to find the value after temp
        )
    )

BORIS, need to explain the SetValue and ValueOf Links in Chapter 2.  It fits with the "optimization" section

.. code-block:: scheme

    (Define
        (DefinedSchemaNode "list_of_n_primes")
        (Lambda
            (VariableNode "n")

        )
    )

    




BORIS
Look at explaining DefinedPredicateNode


BORIS. Check out the https://github.com/opencog/atomspace/blob/master/examples/pattern-matcher/type-signature.scm example.  
BORIS SignatureLink and DefinedTypeNode
Let's start with data structures.  In C, for example, there is the :c:`struct` keyword, to declares a collection of variables that are packaged up together as a unified code object.














NEXT CHAPTER BEGINS SOON.  BORIS YELTSIN




Intro.
We will also cover the difference between the execution and the evaluation context.

We've gotten a lot of mileage out of :code:`cog-execute!`, but BORIS YELTSIN

So we saw above how we could use :code:`cog-evaluate!` to evaluate a atom to generate a TruthValue.
But how do we utilize that result to control what our program does next?
In other words, what are the Atomese equivalents for program-flow constructs like If-Then statements, Case statements, etc.?




LP: See if I can get the AndLink stuff to work for partial conditionals, testing it with the side-effect-full eval path from the recursive-loop.scm example


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

Using fuzzy logic, we can define a set of all tall people, and then a person with a height of 175cm could have a 50% membership in that set.
In traditional set theory, an object or data point either belongs or doesn't belong in a set, based on the set membership function.  In other words, traditional sets always have a crisp boundary.  In fuzzy logic, the membership function returns a value between 0 and 1, so there can be a continuous transition from outside the set to inside the set.

But consider the conceptual difference between our statement about Charlie and the statement "The train from Birmingham arrives every day at 10:42am."  Given the legendary unreliability of the London Midland train service, you'd certainly assign that statement a low truth value.
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


BORIS HERE


Now, we want to put him into a "Big Dog" or a "Small Dog" set, depending on his weight.
But first, we need to define a predicate that will evaluate to true if his weight is above a threshold.


BORIS Unnatural Break

So unlike the other query link types, :code:`SatisfactionLink` is appropriate to use in an evaluation context rather than in an execution context.  In fact, 


Let's stop here, and just evaluate our new predicate.

.. code-block:: scheme

    (cog-evaluate! fido_is_big?)

You should get back :scheme:`(stv 0 1)`, aka false.  Fido is not heavier than 15kg.  If you're not convinced, try tweaking Fido's weight or the predicate to get the answer you want.

BORIS Unnatural Break

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


BORIS (CAN I DEFINE MY OWN TYPES, from an atom-uniqueness standpoint???)
BORIS Next Chapter
We'll also talk about the FFI, like using ExecutionOutput and GroundedSchema, or GroundedPredicate, look at "execute.scm"





BORIS.  Understand how Values become Atoms sometimes...  A clue is dropped in the documentation on SleepLink https://wiki.opencog.org/w/SleepLink
He says "NumberNodes are problematic for the AtomSpace".  It appears that numeric values can exist temporarily, and under certain situations then crystalize into nodes.  Hippo has something similar.
