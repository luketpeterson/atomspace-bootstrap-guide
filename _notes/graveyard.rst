

Filtering Sets with PutLink
------------------------------------------------------------------------

So far, all of our queries have been run against the entire Atomspace, but what if want to run the query against just a subset of atoms?
We might want to do this for performance reasons, or perhaps because we wish to iteratively apply filters or other operations to continually refine our set.

Let's start out using :code:`GetLink` to create a superset of all atoms that have a :code:`ListLink` linking them to the :scheme:`(Predicate "weight_in_kg")` atom.
This superset will include both Fido and Fluffy.

.. code-block:: scheme

   (define dog_set
      (cog-execute!
         (Get
            (List
               (Variable "dog_node")
               (Predicate "weight_in_kg")
            )
         )
      )
   )

The above snippet just created a :code:`SetLink`.  Take a look:

.. code-block:: scheme

   (display dog_set)

Now let's use :code:`PutLink` to filter the set.

.. code-block:: scheme



   (cog-execute!
      (Put
         (VariableList
            (Variable "dog_node")
         )
         (And
            (State
               (List
                  (Variable "dog_node")
                  (Predicate "weight_in_kg")
               )
               (Variable "dogs_weight_node")
            )
            (GreaterThan
               (Variable "dogs_weight_node")
               (Number 15)
            )
         )
         dog_set
      )
   )




   (cog-execute!
      (Put
         (VariableList
            (Variable "dog_node")
         )
         (And
            (State
               (List
                  (Variable "dog_node")
                  (Predicate "weight_in_kg")
               )
               (Variable "dogs_weight_node")
            )
            (GreaterThan
               (Variable "dogs_weight_node")
               (Number 15)
            )
         )
         dog_set
      )
   )





   (cog-execute!
      (Put
         (VariableList
            (Variable "dog_node")
         )
         (GreaterThan
            (ValueOf (Variable "dog_node") (Predicate "age"))
            (NumberNode 2)
         )
         dog_set
      )
   )



The complete documentation for PutLink is here: BORIS. As you can see, it has many uses in addition to filtering :code:`SetLink` atoms.



   (cog-execute!
      (Get
         (And
            (State
               (List
                  (Variable "dog_node")
                  (Predicate "weight_in_kg")
               )
               (Variable "dogs_weight_node")
            )
            (GreaterThan
               (Variable "dogs_weight_node")
               (Number 10)
            )
            (GreaterThan
               (ValueOf (Variable "dog_node") (Predicate "age"))
               (NumberNode 2)
            )
         )
         (Variable "dog_node")
      )
   )












BORIS This snippet tried to implement a recusrive iterative loop, but is BORKED because of a strange interplay between LambdaLink and PutLink

.. code-block:: scheme

    (State (Concept "counter") (Number 0))

    (DefineLink
        (DefinedSchema "increment_loop_body")
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

                ; If not, increment "counter" by one, and then call ourselves recursively
                (SetLink
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

                    ; Here is the recursive call.  Call ourselves with a higher iterator
                    (ExecutionOutputLink
                        (DefinedSchema "increment_loop_body")
                        (Plus (Variable "iterator") (Number 1))
                    )
                )
            )
        )
    )


    (cog-execute!
        (ExecutionOutputLink
            (DefinedSchema "increment_loop_body")
            (Number 0)
        )
    )


