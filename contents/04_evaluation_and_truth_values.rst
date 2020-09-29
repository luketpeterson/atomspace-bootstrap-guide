.. role:: scheme(code)
   :language: scheme
.. role:: c(code)
   :language: c

.. _04_evaluation_and_truth_values:

Previous Chapter: :ref:`Programming with Atomese <03_atomese_programming>`

========================================================================
Evaluation and Truth Values
========================================================================

Somehow, we've managed to get pretty far without talking much about what it means to evaluate a predicate expression.

This is a big topic, but it is key to understanding how to use the Atomspace effectively.
Personally found this very confusing initially, so I'll try to build it up bit by bit.

Hopefully we have enough foundation at this point that it'll make more sense for you than it did for me at first.

Evaluating Predicate Expressions
------------------------------------------------------------------------

We've gotten a lot of mileage out of :code:`cog-execute!`, but sometimes it doesn't do what we need.
:code:`cog-evaluate!` is another similar operation, but instead of *executing* the atom, it is used to *evaluate* an atom forming a predicate expression.
:code:`cog-evaluate!` will always return a *TruthValue*, but :code:`cog-execute!` is free to return just about anything at all.

We've already been using predicate expressions as part of other expressions throughout the guide.
I just didn't put a spotlight on it until now.

Consider each of these examples:

.. code-block:: scheme

    (cog-evaluate! 
        (GreaterThan (Number 20) (Number 15))
    )

    (cog-evaluate! 
        (FalseLink)
    )

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

Each one of those examples above, and all other predicate expressions, can be evaluated to a single TruthValue; True, False, or somewhere in between.

EvaluationLink to Assert Stuff with PredicateNode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:code:`EvaluationLink` is the central mechanism that allows arbitrary predicates to be evaluated.
It has two important roles where it takes on what feel to me like two disparate functions.
Personally, I don't know why separeate atom types weren't used.  Perhaps somebody with more expertise can shed some light on that question.

Anyway, the first role of :code:`EvaluationLink` is to associate a TruthValue with a :code:`PredicateNode` expression.

Very early in the `Atomspace examples <https://github.com/opencog/atomspace/blob/master/examples/atomspace/knowledge.scm>`_, :code:`EvaluationLink` pops up without any introduction.
It says this:

.. code-block:: scheme

    (EvaluationLink
        (PredicateNode "_obj")
        (ListLink
            (ConceptNode "make")
            (ConceptNode "pottery")))

The first thing to understand is that we are making an *assertion* here.
We are telling the Atomspace that :scheme:`(Concept "make")` and :scheme:`(Concept "pottery")` are valid, aka *true*, arguments for :scheme:`(Predicate "_obj")`.
In English, the above statement should be read as "Pottery can be Made.", or more pedantically, "**Pottery** is a valid **Object** for **Make**".

So, when the first argument to :code:`EvaluationLink` is a :code:`PredicateNode` and the entire expression is grounded, this is how we *declare* that a predicate expression is true, for some given arguments.

And once we've declared it, we can query for it, just like any other relationship in the Atomspace.

Can we make pottery?  Let's ask:

.. code-block:: scheme

    (cog-evaluate!
        (SatisfactionLink
            (AndLink
                (EvaluationLink
                    (PredicateNode "_obj")
                    (ListLink
                        (ConceptNode "make")
                        (Variable "predicate_object")
                    )
                )
                (EqualLink
                    (ConceptNode "pottery")
                    (Variable "predicate_object")
                )
            )
        )
    )

Yes, it seems we can.

Can we make rain?  Let's see...

.. code-block:: scheme

    (cog-evaluate!
        (SatisfactionLink
            (AndLink
                (EvaluationLink
                    (PredicateNode "_obj")
                    (ListLink
                        (ConceptNode "make")
                        (Variable "predicate_object")
                    )
                )
                (EqualLink
                    (ConceptNode "rain")
                    (Variable "predicate_object")
                )
            )
        )
    )

Unfortunately, no. :-(

DefinedPredicateNode to Create Predicate Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To understand the second role of :code:`EvaluationLink`, we should talk about defining predicate functions whose TruthValue is a function of the arguments passed in.

In the previous chapter, we looked at the :code:`DefinedSchemaNode` to name a function or subroutine.
For predicates, there are some advantages to using :code:`DefinedPredicateNode` instead.

Just like :code:`DefinedSchemaNode`, :code:`DefinedPredicateNode` is a naming node type that can be ued with a :code:`DefineNode`.
But :code:`DefinedPredicateNode` tells the Atomspace that the function in question can be evaluated to a :code:`TruthValue`.

Also, you may use a :code:`LambdaLink` as the body of a :code:`DefinedPredicateNode`, but it's not required unless you want to pass arguments.

So, let's rewrite the :code:`SatisfactionLink` query above using a :code:`DefinedPredicateNode`.

.. code-block:: scheme

    (DefineLink
        (DefinedPredicateNode "can_be_made?")
        (LambdaLink
            (Variable "test_object")
            (SatisfactionLink
                (Variable "predicate_object")
                (AndLink
                    (EvaluationLink
                        (PredicateNode "_obj")
                        (ListLink
                            (ConceptNode "make")
                            (Variable "predicate_object")
                        )
                    )        
                    (EqualLink
                        (Variable "test_object")
                        (Variable "predicate_object")
                    )
                )
            )
        )
    )

So there's our predicate function.  Notice that we need to declare both :code:`VariableNode` atoms now,
one gets its value from the :code:`LambdaLink` as an argument and the other gets its value as part of the query in :code:`SatisfactionLink`.
If we don't do this, the Atomspace has no way of knowing which :code:`VariableNode` is which.

How do we call it?

.. code-block:: scheme

    (cog-evaluate!
        (Evaluation
            (DefinedPredicate "can_be_made?")
            (ConceptNode "pottery")
        )
    )

This is the other role of :code:`EvaluationLink` I was talking about.
Here :code:`EvaluationLink` wraps the dispatch of a :code:`DefinedPredicateNode`, in the same way that we used :code:`ExecutionOutputLink` in the previous chapter.

.. note:: QUESTION FOR SOMEONE SMARTER THAN ME.  Is there a cleaner and more idiomatic way to evaluate EvaluationLink assertions???  The documentation for EvaluationLink defines a "LessThan" PredicateNode.  But is there a way to call that Predicate without a SatisfactionLink???  Even if it doesn't function like a true numeric LessThan, and only knows about the number pairs that have been asserted.

SequentialAndLink & SequentialOrLink Flow Control
------------------------------------------------------------------------

Last chapter, we introduced :code:`CondLink` for program flow management.
:code:`CondLink` operates in an execution context, to execute one atome vs. another depending on the outcome of evaluating a predicate expression.
If you're operating in an evaluation context, e.g. in the process evaluating a predicate, then logical *And* & *Or* operators can dictate your program flow.

To elaborate: consider the statement "The car must stop if the light is yellow or red."
When evaluating the "Light is yellow or red" expression to determine if the car must stop, and "Light is yellow" is true, there is no need to evaluate "Light is red".
You already know the expression is true.

"And" expressions have the same property for false components.  If you are looking for someone "Tall and Handsome", you can stop if you find that one criteria is not met and there is no need to evaluate the other.

The implication of statements composed of logical "And" and "Or" operators is that it is irrelevant to the outcomem, whether or not one branch of the expression is evaluated if another branch has already determined the outcome.

Atomese takes this one step further with the :code:`SequentialAndLink` & :code:`SequentialOrLink` atoms.
They provide the guarantee that branches will be evaluated in sequence and that the evaluation will stop once the outcome of the whole expressions is conclusively determined.

Therefore, :code:`SequentialAndLink` and :code:`SequentialOrLink` can be used as general purpose flow control primitives from which you can construct complex predicate expressions.

This works even in situations where evaluating one branch of the expression may produce a side effect, such as the examples from RobotOS, cited in the documentation.

In addition, :code:`SequentialAndLink` & :code:`SequentialOrLink` can take an argument list of arbitrary length, unlike their strictly binary logical peers in other languages.

So, in summary, :code:`SequentialAndLink` will evaluate each branch in sequence unless a branch evaluates to false, in which case it will cease evaluating subsequent branches and will return false.
If all branches evaluate to true, the :code:`SequentialAndLink` itself will evaluate to true.

:code:`SequentialOrLink` is similar, but will continue evaluation as long as each branch evaluates to false, and will terminate evaluation after evaluating the first branch that evaluates to true.
:scheme:`(SequentialOrLink A B C)` is equivalent to :scheme:`(Not (SequentialAndLink (Not A) (Not B) (Not C)))`.

IsPrime? An Example in Atomese Code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pulling together many of the concepts we've covered up to this point, here is a predicate expression to determine whether a number is prime.

But first, one of the building blocks of that is a function to determine if a number is a positive integer, or a real number that is not a positive integer.
The function below checks this using a "level-crossing" algorithm.  In other words, all positive integers will eventually equal 1 if 1 is subtracted from them an arbitrary number of times.
I admit this is not a good solution for many many reasons, but Atomese gave me a limited palette of primitive operations to work with. 

.. code-block:: scheme

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

    ; Test our "is_pos_integer?" function
    (cog-evaluate!
        (Evaluation
            (DefinedPredicate "is_pos_integer?")
            (Number 2)
        )
    )

So, with our :code:`is_pos_integer?` function implemented above, we can create our :code:`is_prime?` function.
We need to split our funtion into two parts to facilitate recursion, following the same pattern we used last chapter for our :code:`fibonacci_iterative` function.

.. code-block:: scheme

    (Define
        (DefinedPredicateNode "is_prime_helper")
        ; Determines whether "x" is evenly divisible by "i" or another integer greater than "i"
        ; In otherwords, returns partial NOT prime.  Intended to be called by "is_prime?"
        ; If called with i=2, false means "x is prime", true means "x is not prime"

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

    ; Test our "is_prime_helper" function
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

    ; Test our "is_prime?" function
    (cog-evaluate!
    (Evaluation
        (DefinedPredicate "is_prime?")
        (Number 37)
        )
    )

.. note::

    QUESTION FOR SOMEONE SMARTER THAN ME.  What's the best "chaining" link for the execution context??? i.e. what link type should I use to say "first execute this, then that, then this other thing"?

    I've tried with :code:`ListLink` and :code:`SetLink` and the results have been mixed.  It seems to work in some situations, but in other it seems to go horribly wrong.

    This below works, but then if I put it into a LambdaLink instead of a "raw" Define then it stops working.  What's the right way to do this instead?

    .. code-block:: scheme

        (DefineLink
            (DefinedSchemaNode "make_nighttime")
            (ListLink
                (PutLink
                    (State
                        (Variable "switch_var")
                        (Concept "On")
                    )
                    (Concept "Moonlight")
                )
                (PutLink
                    (State
                        (Variable "switch_var")
                        (Concept "Off")
                    )
                    (Concept "Sunlight")
                )
            )
        )

























BORIS, Include discussion about FFI, like a printf debug funcrtion

BORIS look at "execute.scm"

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


    


BORIS. Check out the https://github.com/opencog/atomspace/blob/master/examples/pattern-matcher/type-signature.scm example.  
BORIS SignatureLink and DefinedTypeNode
Let's start with data structures.  In C, for example, there is the :c:`struct` keyword, to declares a collection of variables that are packaged up together as a unified code object.

BORIS (CAN I DEFINE MY OWN TYPES, from an atom-uniqueness standpoint???)

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









The Philosophy of Truth
------------------------------------------------------------------------

When you run that :code:`cog-evaluate!` snippet above, you should get this:

.. code-block:: scheme

    (stv 1 1)

"stv" in this case stands for *Simple Truth Value*, and an STV is composed of two floating point numbers: *Strength* and *Confidence*.
In our case, they are both exactly 1.  The expression was 100% true, and we are 100% sure of that.

BORIS introduce StrengthOf & CondfidenceOf

BORIS, include the fact that a truthValue is attached to an atom with a special key.  Explained in values.scm example.

BORIS PredicateFOrmula as a way to compose Truth Values
BORIS, look at PredicateFormula, it Constructs a TruthValue from two number values

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





LP: See if I can get the AndLink stuff to work for partial conditionals, testing it with the side-effect-full eval path from the recursive-loop.scm example

BORIS, talk about how both sides can potentially execute, and it's just up to the end to decide which side to use.  How there isn't a program counter, as in precedural programming.

Boris, what happens if something has a truth value of 0.5???  Which link is created???  Both.


BORIS YELTSIN
Talk about side-effect-free vs. side-effects, SequentialAndLink



BORIS.  Explain AnchorNodes??









BORIS.  Understand how Values become Atoms sometimes...  A clue is dropped in the documentation on SleepLink https://wiki.opencog.org/w/SleepLink
It says "NumberNodes are problematic for the AtomSpace".  It appears that numeric values can exist temporarily, and under certain situations then crystalize into nodes.  Hippo has something similar.
