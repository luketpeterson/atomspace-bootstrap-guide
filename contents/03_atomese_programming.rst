.. role:: scheme(code)
   :language: scheme
.. role:: c(code)
   :language: c

.. _03_atomese_programming:

Previous Chapter: :ref:`Structured Knowledge <02_representing_knowledge>`

========================================================================
Programming with Atomese
========================================================================

In the previous chapter, we introduced Atomspace queries and a little bit about the execution model with :code:`cog-execute!`.
Now we'll go deeper into some more of the program-flow constructs that allow Atomese to behave like a complete programming language.

In this upcoming chapter, we'll deal with Atomese approaches to conditionals, code factoring (i.e. functions), and looping.

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
There is a lot to say about TruthValues, and we'll get there soon.  For now you can think of them as Boolean True/False or Yes/No values, just know that there is a lot more to them.

.. note:: :code:`SatisfactionLink` is actually the basic building-block from which all of the other query link types are constructed.

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

This :code:`PutLink` expression appears fairly simple but some of the behavior is a bit subtle and non-obvious, and it tripped me up badly at first.
Understanding :code:`PutLink` is critical to internalizing a key Atomspace concept, i.e. learning how to think about atoms that represent data aka program state vs. atoms that represent operations that can affect the data.
In another programming language we would call these "data" and "code".

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
So why do we need :code:`PutLink` then?

Ungrounded Expressions can Represent "Latent Code"
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

A clue to what is happening can be found by dropping the :code:`cog-execute!`, and just observing what happens when the :code:`CondLink` and all its referenced atoms are added to the Atomspace.
Notice that the :code:`StateLink` connecting :scheme:`(Predicate "conditional_result")` to :scheme:`(Concept "Yes")` is gone altogether.
Remember that the act of creating a :code:`StateLink` removes a prior extant :code:`StateLink` with the same target.

This left me scratching my head for a minute.  I was stuck in the mindset that I was just inputting code, and the :code:`StateLink` behavior would only be triggered when the code actually executed.  Wrong!

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

Now I'll add a bit more nuance.  You can also think of an ungrounded expression as a **Hypothetical** or **Abstract** statement.
As long as an expression is ungrounded, it doesn't really say anything specific or concrete.

So in terms of the example, the :code:`StateLink` atoms we want to input say "Connect *something* with 'Yes'" and "Connect *something* with 'No'".
But without that *something* being a specific concrete atom, the :code:`StateLink` can't do its behavior to ensure uniqueness and thus both :code:`StateLink` atoms are allowed to exist at the same time.

That's where :code:`PutLink` comes in.  When :code:`PutLink` is executed, the ungrounded expression is grounded using the atom(s) supplied to :code:`PutLink`.
In the case of the example, that is :scheme:`(Predicate "conditional_result")` .  And the atoms of the newly grounded expression are put into the Atomspace.

Grounding a DeleteLink Removes an Atom
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We just saw how grounding a :code:`StateLink` can cause another conflicting :code:`StateLink` atom to be deleted if they share the same target atom.
This is a special case of a general behavior.  To illustrate that, I'll introduce the :code:`DeleteLink` atom.

As long as the :code:`DeleteLink` remains ungrounded, it doesn't to anything.
But the instant it is grounded, it ceases to exist, and takes whatever atoms it references out along with it.

Consider this Scheme snippet:

.. code-block:: scheme

    (define hello (Concept "Here I am!"))

    (define delete_me
        (PutLink
            (DeleteLink
                (Variable "the_concept")
            )
            hello
        )
    )

We just defined two scheme symbols: :code:`hello` and :code:`delete_me`.  Let's prove :code:`hello` exists by having a look.

.. code-block:: scheme

    (display hello)

Yup.  Just where we left it.
We can also look at :code:`delete_me`, and we will see it contains an ungrounded :code:`DeleteLink` expression.

But what happens when we execute :code:`delete_me`?

.. code-block:: scheme

    (cog-execute! delete_me)

Well... We just annihilated the atom referenced by :code:`hello`.  To prove it, we'll try to display it again.

.. code-block:: scheme

    (display hello)

So sad. :-(  We now get :code:`#<Invalid handle>`.

Of course, deleting a pre-specified atom using a :code:`DeleteLink` inside a :code:`PutLink` isn't really any more useful than just declaring the :code:`DeleteLink` directly, and skipping all the :code:`PutLink` machinations.
We could have just done this:

.. code-block:: scheme

    (cog-execute! (DeleteLink hello))

However, next, we'll cover how to query for atoms inside a :code:`PutLink`, so we'll be able to do things like deleting all atoms returned by a query, for example.

At some point, I recommend exploring the Atomspace execution model further by going through the `"assert-retract.scm" OpenCog example <https://github.com/opencog/atomspace/blob/master/examples/atomspace/assert-retract.scm>`_.
In particular, understanding the mechanics of :code:`PutLink` and :code:`DeleteLink` will help you understand what really happens when you invoke :code:`cog-execute!`.

Finding Atoms with a Query Inside a PutLink
------------------------------------------------------------------------

To implement any complex behavior beyond the trivial toy examples we've seen so far,
like our conditional above that branched based on a constant, we need to operate on data from the Atomspace.

Consider a simple counter that might be written in C like this:

.. code-block:: c

    int counter = 0;

    counter = counter + 1;

How would that look in Atomese?

.. code-block:: scheme

    (State (Concept "counter") (Number 0))

    (cog-execute!
        (PutLink
            (State
                (Concept "counter")
                (PlusLink
                    (Variable "our_num")
                    (Number 1)
                )
            )
            (MeetLink
                (State
                    (Concept "counter")
                    (Variable "our_num")
                )
            )
        )
    )

As you can see in both C and Atomese, we begin by declaring :code:`counter` and initially setting it to 0 (Zero).
In a way, a declaration with an initial value is like an assignment, but you couldn't write much of a program using only declarations in C, and that's also true in Atomese.

So, looking at the interesting part of the example, we supplied two atoms to :code:`PutLink`.
We can think of the first argument as being like the "R-Value" and the second argument as being like the "L-Value".
That'll mean something if you've spent significant time working through gcc errors.
If not, consider yourself fortunate for avoiding that particular drain of life energy.

Basically, the first argument is the expression to the right of the equal-sign in the assignment.
From our C example, that would be :c:`counter + 1`.  Think of the R-Value as "what" is being assigned.

The second argument to :code:`PutLink` is the part of the assignment to the left of the equal side.
From our C example, that would be :c:`counter`.  Think of the L-Value as "where" you are assigning to.

Again, this is a flawed comparison with the assignment operation in C, so don't try and stretch the analogy too far.
For example, you could change the ConceptNode in the first expression, so a totally different atom would be created by executing the :code:`PutLink`

Coming full circle, :code:`QueryLink` and :code:`BindLink` are actually implemented using PutLink.
The increment example above is functionally equivalent to:

.. code-block:: scheme

    (cog-execute!
        (QueryLink
            (State
                (Concept "counter")
                (Variable "our_num")
            )
            (State
                (Concept "counter")
                (PlusLink
                    (Variable "our_num")
                    (Number 1)
                )
            )
        )
    )

The `"get-put.scm" OpenCog example <https://github.com/opencog/atomspace/blob/master/examples/atomspace/get-put.scm>`_
Further explores :code:`PutLink` and demonstrates exactly how a :code:`BindLink` can be composed from a :code:`GetLink` and a :code:`PutLink`.  
I recommend going through that example as well as the `"bindlink.scm" example <https://github.com/opencog/atomspace/blob/master/examples/atomspace/bindlink.scm>`_.

Factoring and Functions in Atomese
------------------------------------------------------------------------

Up until now, we've been using Scheme's :scheme:`(define)` mechanism to as a way to get a symbolic reference to an atom we intend to use later.
But this mechanism has some limitations that we're about to cover, not to mention its reliance on Scheme.
Remember the Atomspace can theoretically be accessed from other non-Scheme environments.

In this section we're going to introduce some code segmentation and organization primitives that are native to Atomese.

Let's being with Atomese's own version of :code:`define`, the :code:`DefineLink`.  Here's an example:

.. code-block:: scheme

    (Define (DefinedSchemaNode "five") (NumberNode 5))

We just introduced two new atoms.  :code:`DefineLink` gives a name to something else.  That's it.

The first argument to a :code:`DefineLink` needs to be a "naming" node.
There are 3 special "naming" node types: :code:`DefinedSchemaNode`, :code:`DefinedPredicateNode`, and :code:`DefinedTypeNode`.
We'll cover the latter two in due course, but here we'll focus on :code:`DefinedSchemaNode`.
The second atom provided to :code:`DefineLink` is the definition body.  In our case, it is just a simple :code:`NumberNode`.

:code:`DefinedSchemaNode` is the most general of the "naming" node types.  A :code:`DefinedSchemaNode` can give a name to any other atom.

We can now use our :code:`DefinedSchemaNode` as we would use the atom it represents...  Almost.
This example below does exactly what you think it should do.

.. code-block:: scheme

    (cog-execute!
        (State
            (Concept "counter")
            (DefinedSchemaNode "five")
        )
    )

But without the :code:`cog-execute!`, the :code:`StateLink` connects :scheme:`(Concept "counter")` to :scheme:`(DefinedSchemaNode "five")`, and not to :scheme:`(NumberNode 5)`.
The :code:`DefinedSchemaNode` will be replaced by the node it represents, but only in an execution context.

Basic Subroutines
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:code:`DefinedSchemaNode` can also be used to define subroutines.  By subroutine I mean that there are no function arguments and no return values.
The expectation is that all communication to and from the subroutine happens by way of global program state.
Subroutines are just a way to dispatch one chunk of code from a different place in the program.
Most modern programming languages don't even have subroutines because they can lead to hideous spaghetti code and functions with arguments reduce to subroutines in the degenerate case.
If you've written assembly code by hand or worked with a very old language like `Integer BASIC <https://en.wikipedia.org/wiki/Integer_BASIC>`_, you'll certainly appreciate why subroutines are insufficient to architect a complex piece of software. 

All that said, subroutines are simpler than functions so let's start there.
Here is an example:

.. code-block:: scheme

    (DefineLink
        (DefinedSchemaNode "turn_on_switch")
        (PutLink
            (State
                (Variable "switch_placeholder")
                (Concept "On")
            )
            (Concept "Global Switch")
        )
    )

We can call it like this:

.. code-block:: scheme

    (cog-execute! (DefinedSchemaNode "turn_on_switch"))

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

LambdaLink Lets you Pass Function Arguments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The difference between subroutines vs. procedures and functions are the arguments that can be passed in and out.
:code:`LambdaLink` is the mechanism for defining functions in Atomese, and specifying the arguments that can be passed in.

Here is an example function that squares the incoming :code:`NumberNode` argument:

.. code-block:: scheme

    (DefineLink
        (DefinedSchemaNode "square")
        (LambdaLink
            (VariableNode "x")
            (TimesLink
                (VariableNode "x")
                (VariableNode "x")
            )
        )
    )

So there's our function.  It takes a :code:`NumberNode` and squares it.  Now, how do we call it?

Well, unfortunately we can't just :code:`cog-execute!` it like the simple subroutine.  That will cause an error.
The reason is a little bit convoluted, but in essence, we need a seperate operation to pack up the arguments and bind them to the :code:`VariableNode` atoms used inside the fucntion, and then dispatch the function execution.
That "dispatch" process is handled by the :code:`ExecutionOutputLink` atom.
Atomese is very "assembly-language-like", so very little is magically done for you, as might happen in higher-level languages.

We call it like this:

.. code-block:: scheme

    (cog-execute!
        (ExecutionOutputLink
            (DefinedSchemaNode "square")
            (NumberNode 2)
        )
    )

VariableList and Typed Variables
------------------------------------------------------------------------




BORIS VariableList
BORIS TypedVariables


BORIS
Look at explaining DefinedSchemaNode (look at the PutLink OpenCog web documentation for ideas) and DefinedPredicateNode


BORIS. Check out the https://github.com/opencog/atomspace/blob/master/examples/pattern-matcher/type-signature.scm example.  
BORIS SignatureLink and DefinedTypeNode
Let's start with data structures.  In C, for example, there is the :c:`struct` keyword, to declares a collection of variables that are packaged up together as a unified code object.








Looping with Tail Recursion
------------------------------------------------------------------------

BORIS






NEXT CHAPTER BEGINS SOON.  BORIS YELTSIN




Intro.
We will also cover the difference between the execution and the evaluation context.


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