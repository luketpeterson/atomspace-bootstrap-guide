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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
Therefore, it's more accurate to say the second argument matches existing atoms, providing concrete meanings to the :code:`VariableNode` atoms in the first argument,
While the first argument is a template for the new atoms to insert into the Atomspace, after that template is grounded in terms of the second argument.

Coming full circle, :code:`QueryLink` and :code:`BindLink` are actually implemented using PutLink.
The increment example above is functionally equivalent to this:

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

Defines, Schemas, Lambdas and Functions in Atomese
------------------------------------------------------------------------

Up until now, we've been using Scheme's :scheme:`(define)` mechanism to as a way to get a symbolic reference to an atom we intend to use later.
But this mechanism has some limitations that we're about to cover, not to mention its reliance on Scheme.
Remember the Atomspace can theoretically be accessed from other non-Scheme environments.

In this section we're going to introduce some code segmentation and organization primitives that are native to Atomese.

Let's being with Atomese's own version of :code:`define`, the :code:`DefineLink`.  Here's an example:

.. code-block:: scheme

    (Define (DefinedSchema "five") (NumberNode 5))

We just introduced two new atom types: :code:`DefineLink` and :code:`DefinedSchemaNode`.  :code:`DefineLink` gives a name to something else.  That's it.

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
            (DefinedSchema "five")
        )
    )

But without the :code:`cog-execute!`, the :code:`StateLink` connects :scheme:`(Concept "counter")` to :scheme:`(DefinedSchemaNode "five")`, and not to :scheme:`(NumberNode 5)`.
The :code:`DefinedSchemaNode` will be replaced by the node it represents, but only when it is reduced by executing it.
Think of it like a :c:`const` in C, not like a pre-processor :c:`#define`.

Basic Subroutines
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:code:`DefinedSchemaNode` can also be used to define subroutines.  By subroutine I mean a block of code you can call, but where there are no function arguments.
The expectation is that all communication between the subroutine and the caller happens by way of global program state.
Subroutines are just a way to dispatch one chunk of code from a different place in the program.
Most modern programming languages don't even have subroutines because they can lead to hideous spaghetti code and functions with arguments reduce to subroutines in the case where no arguments are needed.
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

That's it.  The :scheme:`(DefinedSchemaNode "turn_on_switch")` takes the place of the whole :code:`PutLink` and all its dependent atoms.

LambdaLink Lets you Pass Function Arguments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The difference between subroutines vs. procedures and functions are the arguments that can be passed.
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

So there's our function.  It takes a :code:`NumberNode` and squares it.
The first argument to :code:`LambdaLink` is :scheme:`(VariableNode "x")`.  This specifies that our function expects one argument, and we're mapping that argument onto :scheme:`(VariableNode "x")`.

Now, how do we call it?

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

The first argument is our :code:`DefinedSchemaNode`, and the second atom argument is the parameter we are passing to our function.

.. note::

    QUESTION FOR SOMEONE SMARTER THAN ME.  I don't fully understand why the below snippet doesn't fully reduce when executed.

    .. code-block:: scheme

        (State (Concept "counter") (Number 2))

        (DefineLink
            (DefinedSchema "increment")
            (LambdaLink
                (VariableList)
                (PutLink
                    (State
                        (Concept "counter")
                        (Plus
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
        )

        (cog-execute!
            (ExecutionOutputLink
                (DefinedSchema "increment")
                (List)
            )
        )
    
    **RESULT** 

    .. code-block:: scheme

        $1 = (StateLink
        (ConceptNode "counter")
        (PlusLink
            (NumberNode "2")
            (NumberNode "1")))

    **EXPECTED RESULT**
    
    .. code-block:: scheme

        $1 = (StateLink
        (ConceptNode "counter")
        (NumberNode "3"))

    Yet, if I remove the LambdaLink & ExecutionOutputLink, it does what I would expect and the PutLink fully reduces.

Typed Variables and VariableLists as Arguments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:code:`LambdaLink` takes a single atom to specify its argument, but what if we want to pass multiple arguments to a function?
Enter the :code:`VariableList` atom.
A :code:`VariableList`, as the name would suggest, is an ordered list to define multiple :code:`VariableNode` atoms, and thus multiple function arguments.

You can use it with a :code:`LambdaLink`, as in the example below, where I've reimplemented simple multiplication using :code:`TimesLink`.

.. code-block:: scheme

    (DefineLink
        (DefinedSchema "multiply")
        (LambdaLink
            (VariableList
                (VariableNode "x")
                (VariableNode "y")
            )
            (TimesLink
                (VariableNode "x")
                (VariableNode "y")
            )
        )
    )

And we use a :code:`ListLink` to pass the arguments, when we want to call it.

.. code-block:: scheme

    (cog-execute!
        (ExecutionOutputLink
            (DefinedSchema "multiply")
            (List
                (Number 2)
                (Number 3)
            )
        )
    )

If your list doesn't have enough arguments to match the :code:`VariableList` of the :code:`LambdaLink` that you are calling, then it will cause an error.
On the other hand, if you pass additional extraneous arguments they will be silently ignored.

.. note:: :code:`VariableList` is one type of :code:`Connector`.  :code:`Connector` atoms are used to declare which atoms may be linked with which other atoms.  They are key to specifying custom grammars, which we will cover later on. 

We have been using "naked" unadorned :code:`VariableNode` atoms, which can map onto any other atom whatsoever, irrespective of the atom's type.
In some situations we may want to constrain the types of atoms that our function accept as arguments.
This is done with a :code:`TypedVariableLink`.

Here is the "multiply" function from above, but using :code:`TypedVariableLink` atoms to declare that the parameters should be :code:`NumberNode` atoms.

.. code-block:: scheme

    (DefineLink
        (DefinedSchema "multiply")
        (LambdaLink
            (VariableList
                (TypedVariableLink
                    (VariableNode "x")
                    (TypeNode "NumberNode")
                )
                (TypedVariableLink
                    (VariableNode "y")
                    (TypeNode "NumberNode")
                )
            )
            (TimesLink
                (VariableNode "x")
                (VariableNode "y")
            )
        )
    )

:code:`TypeNode` is an atom type that represents an atom type.  Whoa... that's meta.
It can refer to any of the built-in atom types we've used so far; in addition, new types can be defined.
In a later chapter we'll get deeper into Signatures and creating new atom types.

.. note:: QUESTION FOR SOMEBODY SMARTER THAN ME.  What is :code:`TypedVariableLink` actually useful for when used in a :code:`LambdaLink`?  It seems to be ignored when executing lambdas.  I see how it gives extra criteria when matching.  Am I missing something?

Finally, I should mention that the variable declaration features of Atomese aren't just for :code:`LambdaLink` and functions.

:code:`VariableList` and :code:`TypedVariableLink` can be used with any of the query link types, such as :code:`MeetLink`, :code:`QueryLink`, :code:`SatisfactionLink`, etc.
Often including a :code:`VariableList` in a match expression is optional, because the variables can be inferred from the expression itself.
When there is any ambiguity, however, including a :code:`VariableList` is required.

In addition, including a :code:`TypedVariableLink` around a :code:`VariableNode` in a query can help narrow down the possible matches that can satisfy the query.

Looping with Tail Recursion
------------------------------------------------------------------------

Atomese doesn't have loops, in the way most iterative programming languages do.  Like most pure functional languages, looping behavior is done through recursion.
To utilize recursion, you make the loop body into a function, and then you have the function call itself.
Every recursive function fundamentally has two paths through it, one path that returns without further iterations, and one path that reduces the problem into a smaller problem and then calls itself to solve that new reduced problem.

It's mathematically proven (By Alan Turing himself) than any algorithm that can be expressed with iteration can also be expressed with recursion.
However, this says nothing about the gymnastics that are needed for the implementation of a particular algorithm.
In any case, this is not the point of the guide and the topic is thoroughly covered elsewhere.

The "Tail" in Tail Recursion means that self-calling is the very last thing the function does before exiting.
Therefore, there is no need to retain the execution state for the "parent" function.
Traditional recursion has a bad reputation for consumption of stack memory, because each iteration allocates a new stack frame.  Tail recursion makes this a non-issue.
In fact, in many programming languages, tail recursion has been shown to be as fast as iterative looping.

.. note:: Atomese is not a language designed for execution speed, so don't expect your Atomese code to be fast compared to anything written in a more traditional compiled language, or even an interpreted scripting language.

Below is an example that uses tail recursion to increment a counter repeatedly.
This could stand-in for a for-loop.

.. code-block:: scheme

    (DefineLink
        (DefinedSchema "loop_body")
        (LambdaLink
            (VariableNode "iterator")
            (CondLink
                ; Check to see if the "iterator" parameter is > 4
                (GreaterThan
                    (VariableNode "iterator")
                    (Number 4)
                )

                ; If so, we are done, 
                (VariableNode "iterator")

                ; Here is the recursive call.  Call ourselves with iterator+1
                (ExecutionOutputLink
                    (DefinedSchema "loop_body")
                    (Plus (Variable "iterator") (Number 1))
                )
            )
        )
    )

So the above function checks to see if the parameter is > 4, and if not, calls itself with its parameter + 1.
This means it will be called 5 times in total.  Of course, it doesn't actually do anything, so there is no point to the function.
It's just a pure illustration of a recursive function implementing a loop.

Here is a *slightly* more useful example, a function to calculate the nth term of the Fibonacci sequence.
Of course this is far from optimal, given the order-n^2 execution difficulty for a sequence that should be order-n or better.

.. code-block:: scheme

    (DefineLink
        (DefinedSchema "fibonacci")
        (LambdaLink
            (VariableNode "n")
            (CondLink
                ; The 1st term is 0, so check to see if n is smaller than 2
                (GreaterThan
                    (Number 2)
                    (VariableNode "n")
                )
                (Number 0)

                ; The 2nd term is 1, so check to see if n is smaller than 3
                (CondLink
                    (GreaterThan
                        (Number 3)
                        (VariableNode "n")
                    )
                    (Number 1)

                    ; Otherwise, the nth term is fibbonacci(n-1) + fibbonacci(n-2)
                    ; Here are the recursive calls.
                    (PlusLink
                        (ExecutionOutputLink
                            (DefinedSchema "fibonacci")
                            (MinusLink (VariableNode "n") (Number 1))
                        )
                        (ExecutionOutputLink
                            (DefinedSchema "fibonacci")
                            (MinusLink (VariableNode "n") (Number 2))
                        )
                    )
                )
            )
        )
    )

So here's one way to improve our solution to only require n iterations as opposed to n^2. 

.. code-block:: scheme

    (DefineLink
        (DefinedSchema "fibonacci_iterative")
        (LambdaLink
            (VariableList
                (Variable "n")
                (Variable "current_iter")
                (Variable "current_term")
                (Variable "previous_term")
            )
            (CondLink
                ; If we're reached the end of the sequence, return "current_term"
                (Not (GreaterThan ; not-greater-than is equivalent to less-than-or-equal-to
                    (Variable "n")
                    (Variable "current_iter")
                ))
                (Variable "current_term")

                ; Call ourselves recursively, to get the next term
                (ExecutionOutputLink
                    (DefinedSchema "fibonacci_iterative")
                    (List
                        (Variable "n")
                        (Plus (Variable "current_iter") (Number 1))
                        (Plus (Variable "current_term") (Variable "previous_term"))
                        (Variable "current_term")
                    )
                )
            )
        )
    )

This appraoch requires us to "seed" our sequence with some earlier terms, but it is considerably more efficient than the earlier solution.
Here is an example of how we might call it:

.. code-block:: scheme

    (cog-execute!
        (ExecutionOutputLink
            (DefinedSchema "fibonacci_iterative")
            (List
                (Number 9) ; We want the 9th term
                (Number 2) ; We're passing in the 2nd term
                (Number 1) ; The 2nd term is 1
                (Number 0) ; The 1st term is 0
            )
        )
    )

.. note:: QUESTION FOR SOMEONE SMARTER THAN ME.  This is where I'd like to say "There is this feature for default arguments", but I didn't find anything like that... Does it exist?

.. note:: QUESTION FOR SOMEONE SMARTER THAN ME.  Are there links that let me do something like execute a Schema against every item in a set?  An alternative to looping, a la MapReduce?  Seems like a natural fit for the Atomspace, but I didn't find anything.  Maybe I wasn't looking in the right place.



