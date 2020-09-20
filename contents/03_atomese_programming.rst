.. role:: scheme(code)
   :language: scheme

.. _03_atomese_programming:

Previous Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`

========================================================================
Programming with Atomese
========================================================================

In the previous chapter, we introduced Atomspace queries and a little bit about the execution model with :code:`cog-execute!`.
Now we'll go deeper into some more of the program-flow constructs that allow Atomese to behave like a complete programming language.

In this upcoming chapter, we'll deal with Atomese approaches to conditionals, code factoring (i.e. functions), and looping.
We will also cover the difference between the execution and the evaluation context.

If you are steeped in procedural programming, like I am, there are things about the Atomspace execution model that will require you to turn your brain inside-out.
On the other hand, if you come from a `Lambda Calculus <https://en.wikipedia.org/wiki/Lambda_calculus>`_ or `Functional Programming <https://en.wikipedia.org/wiki/Functional_programming>`_ background then, lucky you!
This next parts will be a lot easier to wrap your head around.

The Atomspace is different from procedural programming languages insofar as there isn't a program counter as I typically understand it.
You can't "follow" the execution in a sequential fashion the way you might be able to in other languages.
Under the hood, obviously, it's software running on a microprocessor so there has to be sequential instruction-flow at some level, but it's abstracted away from you and trying to follow it is counter-productive to understanding how to effectively use the Atomspace.

Conditional Expressions
------------------------------------------------------------------------

In the previous chapter, we used a :code:`MeetLink` and :code:`QueryLink` to query the "weight_in_kg" of "Fido the Dog",
and used a different query to find all dogs (actually all atoms) heavier than 10kg.
But how can we cause a different action to be taken, depending on whether or not Fido is heavier than 10kg?

More generally, how do we compose a conditional expression in Atomese?

Let's start with a simpler conditional.  We'll use a :code:`CondLink` like this:

.. code-block:: scheme

    (cog-execute!
        (CondLink
            (GreaterThan
                (Number 2)
                (Number 1)
            )
            (Concept "Yes")
            (Concept "No")
        )
    )

When you execute the Scheme snippet above, you will see :scheme:`(ConceptNode "Yes")` because 2 is indeed greater than 1.
Simple enough.  :code:`CondLink` takes either 2 or 3 atoms as arguments.

The first is the *conditional* predicate atom.  Something that will evaluate to true or false.  There is a lot more to say about this later.
For now, just remember that :code:`CondLink` predicates must be 100% true or they will be considered false.

Argument 2 is the *consequent* atom, or as I like to think of it, the expression after the **then** keyword in other languages.  Optionally, for argument 3, you can supply a *default* atom, which is basically the **else** expression, to be executed if the conditional evaluates to false. 

Building on that, let's compare Fido's weight rather than just comparing some constants.  First we need to bring Fido back into the Atomspace (assuming you've cleared things out since last chapter's exercises).

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

Now we need a predicate that will query for Fido's weight, and evaluate to true if he's heavier than 10kg.

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
                    (Number 10)
                )
            )
        )
    )

Earlier I promised I wouldn't drop a new atom or other construct on you without at least attempting to demystify it.  :code:`SatisfactionLink` is yet another query link type.
Fundamentally it's just like :code:`MeetLink`, :code:`GetLink`, :code:`QueryLink`, and :code:`BindLink`.

The main feature that sets :code:`SatisfactionLink` apart is that it evaluates to a TruthValue.  True, aka :scheme:`stv(1, 1)`, if the expression could be matched in the Atomspace, and false, aka :scheme:`stv(0, 1)`, if not.
There is a lot to say about TruthValues, and we'll get there soon.  For now you can think of them as Booleans True/False or Yes/No values, just know that there is a lot more to them.

.. note:: :code:`SatisfactionLink` is actually the basic building-block from which all of the other query links are constructed.

Finally, let's use our new :scheme:`fido_is_big?` predicate in a :code:`CondLink` atom.

.. code-block:: scheme

    (cog-execute!
        (CondLink
            fido_is_big?
            (Concept "Yes")
            (Concept "No")
        )
    )

Executing that should get you a resounding :scheme:`(ConceptNode "Yes")`!

Using PutLink to Modify the AtomSpace 
------------------------------------------------------------------------

Now, let's use the result of our conditional to update some state in the Atomspace.
Recall how, a few chapters ago, we used a :code:`StateLink` to create an exclusive link that can only have one result for a given atom.
Here, we will assign a :code:`StateLink` result depending on a :code:`CondLink` conditional execution.

To do this, we will use :code:`PutLink`.  You can think of :code:`PutLink` as the assignment operator of Atomese, akin to "**=**" or "**:=**" in other languages.
Here in our example, we set the :code:`StateLink` association of :scheme:`(Predicate "conditional_result")` with one of two possible :code:`ConceptNode` atoms.

In reality, the comparison of :code:`PutLink` to the assignment operator is flawed because of enormous fundamental differences between the Atomspace and the traditional precedural programming language execution model.
The `OpenCog PutLink documentation <https://wiki.opencog.org/w/PutLink>`_ more accurately describes :code:`PutLink` as a *Beta Redex*, but without a Lambda Calculus background that didn't connect for me.
So, I found the analogy to assignment to be a useful way of bootstrapping my understanding, not only of :code:`PutLink` itself, but the process of learning about :code:`PutLink` gave me a deeper understanding of the Atomspace as a whole.

.. code-block:: scheme

    (cog-execute!
        (PutLink
            (CondLink
                (TrueLink)
                (StateLink
                    (Variable "result_placeholder")
                    (Concept "Yes")
                )
                (StateLink
                    (Variable "result_placeholder")
                    (Concept "No")
                )
            )
            (Predicate "conditional_result")
        )
    )

As you probably expected, running the Scheme snippet above produces this:

.. code-block:: scheme

    (StateLink
        (PredicateNode "conditional_result")
        (ConceptNode "Yes")
    )

So now the one and only :code:`StateLink` associated with :scheme:`(PredicateNode "conditional_result")` points to :scheme:`(ConceptNode "Yes")`.

.. note:: :code:`TrueLink` and its mirror-twin :code:`FalseLink` are atoms that always evaluate to true (or false).  As used above, it's equivalent to saying "if (true)" in another language, and thus it gives me a concise way to demonstrate the behavior of the :code:`CondLink` atom.

This :code:`PutLink` expression appears fairly simple but there is a lot going on here, and some of it is subtle and non-obvious.
Understanding :code:`PutLink` is critical to internalizing a key Atomspace concept, i.e. learning how to think about atoms that represent data vs. atoms that represent transformations and operations that can affect the data.
In another programming language, we would call that "code".

Remember both kinds of atoms live in the Atomspace, and there isn't a simple rule about whether an atom is "data" or it's "code".  Often it can feel like everything is all mixed together.
This is a source of tremendous flexibility, but remember, with great power comes great responsibility ;-)

So back to :code:`PutLink`.  Like the name suggests, it "Puts" atoms into the Atomspace.
The simplest valid :code:`PutLink` I could compose looks like this:

.. code-block:: scheme

    (cog-execute!
        (PutLink
            (Variable "atom_placeholder")
            (Concept "The Atom We Are Putting In")
        )
    )

But hold on...  What's the point?  Isn't that identical to just: :scheme:`(Concept "The Atom We Are Putting In")`?

Yes.  Yes it is.  :code:`PutLink` is not the only way to put atoms into the Atomspace.
We've been putting atoms in the Atomspace since the very first example in this guide.
Why do we need :code:`PutLink` then?

Ungrounded Expressions to Defer Execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's look at what's wrong with this naïve attempt to set our result :code:`StateLink` from the first example.

.. code-block:: scheme

    (cog-execute!
        (CondLink
            (TrueLink)
            (StateLink
                (Predicate "conditional_result")
                (Concept "Yes")
            )
            (StateLink
                (Predicate "conditional_result")
                (Concept "No")
            )
        )
    )

Try it!  Did it do what you expected?  I know I was puzzled when I encountered this behavior.  Actually "annoyed" and "frustrated" are more accurate words.

A clue to what is happening can be found by dropping the :code:`cog-execute!`, and just observing what happens when those atoms are added to the Atomspace.
Notice that the :code:`StateLink` connecting :scheme:`(Predicate "conditional_result")` to :scheme:`(Concept "Yes")` is gone altogether.
Remember that the act of creating a :code:`StateLink` removes a prior extant :code:`StateLink` with the same target.

I was stuck in the mindset that I was just inputting code, and the :code:`StateLink` behavior would only be triggered when the code actually executed.  Wrong!
As I inputted the first arm of the :code:`CondLink`, I actually assigned the result to "Yes" by creating the :code:`StateLink`.
Then, when I inputted the second arm, I re-assigned the result to "No", by creating the new :code:`StateLink`, thus triggering the old one to be destroyed.

Finally, when I executed the :code:`CondLink`, there was no "else" clause atom, because the "then" clause atom had disappeared and so the former "else" clause atom became argument 2, and was interpreted as the "then" clause atom.
So the atom I actually executed looked like this:

.. code-block:: scheme

    (CondLink
        (TrueLink)
        (StateLink
            (PredicateNode "conditional_result")
            (ConceptNode "No")
        )
    )

Basically :code:`if (true) { result = No; }`.  Oops.

So what's the solution?

We introduced ungrounded vs. grounded expressions last chapter when discussing queries.
I remember saying (typing) "Think of a grounded expression as a statement and an ungrounded expression as a question."

Now I'll add a bit more nuance.  You can also think of an ungrounded expression as a **Hypothetical**.
As long as an expression is ungrounded, it is abstract.

So in terms of the example, the :code:`StateLink` atoms we want to input say "Connect *something* with 'Yes'" and "Connect *something* with 'No'".
But without that *something* being a specific concrete atom, the :code:`StateLink` can't do its behavior to ensure uniqueness and both :code:`StateLink` atoms are allowed to exist at the same time.

That's where :code:`PutLink` comes in.  When :code:`PutLink` is executed, the ungrounded expression is grounded using the atom(s) supplied to :code:`PutLink`.
:scheme:`(Predicate "conditional_result")` in the case of the example.  And the atoms of the newly grounded expression are added to the Atomspace.




BORIS, below is probably bunk and should be deleted.

So, looking at our example above, we supplied two atoms to :code:`PutLink`.  First we gave it our :code:`CondLink` and second we supplied :scheme:`(Predicate "conditional_result")`.
In this case, we can think of the first argument as being like the "R-Value" and the second argument as being the "L-Value".
That'll mean something if you've ever spent significant time working through gcc errors.

If not, think of the first argument atom as being the "what" you want to assign, and the second argument atom as being the "where" you want to assign it.

Consider this :code:`PutLink` example:

.. code-block:: scheme

    (cog-execute!
        (PutLink
            (PlusLink
                (Variable "our_num")
                (Number 1)
            )
            (Number 1)
        )
    )






BORIS, Explain how PutLink args are the right and left side of the assignment expression.
So, 

But haven't I been doing asignments already?  What about Fido's weight?
The best analogy in other languages is the way some languages let you declare a variable with an initial value.
In a way, it is an assignment, but you often can't use the exact same syntax to change the value during runtime.

Talk about all the assignments we thought we were doing and how they were just declarations.


BORIS, go through why the trivial approach doesn't work.  i.e. why adding atoms as part of the conditional clobbers the moon

To illustrate this point, what if we were to try this:

.. code-block:: scheme

    (cog-execute!
        (CondLink
            (GreaterThan
                (Number 2)
                (Number 1)
            )
            (QuoteLink
                (StateLink
                    (Predicate "conditional_result")
                    (Concept "Yes")
                )
            )
            (QuoteLink
                (StateLink
                    (Predicate "conditional_result")
                    (Concept "No")
                )
            )
        )
    )



BORIS Finally, grab the example assert-retract, get-put, whatever



BORIS, cover DeleteLink

BORIS, go on and move some of the earlier dicsussion about PutLink to here.








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


BORIS VariableList, Typed Variables (CAN I DEFINE MY OWN TYPES???)
BORIS Next Chapter, program segmentation, DefineLinks, Tail Recursion, etc. look at the recursive-loop.scm example.
We'll also talk about the FFI, like using ExecutionOutput and GroundedSchema, or GroundedPredicate, look at "execute.scm"