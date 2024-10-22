\title Teaching Pure LP with Prolog and a Fair Search Rule

@comment{Small kludge to start already in slides mode (needs to be improved)}
```ciao_runnable
%! \begin{jseval}
async(pgcell) => { pgcell.pgset.main_doc.set_slide_mode(true); }
%! \end{jseval}
```

- **Manuel V. Hermenegildo** ¹ ²
- **Jose F. Morales** ¹ ²
- **Pedro Lopez-Garcia** ² ³

@p ¹ Universidad Politécnica de Madrid (UPM) 
@p ² IMDEA Software Institute 
@p ³ Spanish Council for Scientific Research (CSIC)

@comment{Test of injection of arbitrary HTML (needs an interface of course)}
```ciao_runnable
%! \begin{jseval}
async (pg) => {
  let e = pg.preview_el;
  e.innerHTML = `
Thanks to David S. Warren and Daniela Ferreiro.
<BR>
<h2>Prolog Education Workshop @ ICLP. October 13, 2024. Dallas</h2>
  `;
  pg.show_preview(true);
  pg.update_inner_layout();
}
%! \end{jseval}
```





@comment{
@begin{note}

This presentation corresponds to the paper "Teaching Pure LP with
Prolog and a Fair Search Rule," in the [Prolog Education Group's (EG
2.0) 2024
Workshop](https://prolog-lang.org/Education/PrologEducationWS2024.html)
at [ICLP 2024](https://www.iclp24.utdallas.edu/)

See our [**Course material on logic and constraint logic
programming**](http://www.cliplab.org/logalg/) for more details.

@end{note}
}

# Intro 

- Many ideas on how to best teach Prolog in:

  - [Some Thoughts on How to Teach Prolog](https://cliplab.org/papers/TeachingProlog-PrologBook.pdf), and
  - [Teaching Prolog with Active Logic Documents](http://cliplab.org/papers/ActiveLogicDocuments-PrologBook.pdf) (ALDs)

  (in the "Prolog: The Next 50 Years" book).

@p

@comment{
- Stemming from our experience: 

  - Teaching
  
     - mostly to CS undergrads. 
     - U.T. Austin, U. of New Mexico, T.U.Madrid (UPM)

  - As developers of Ciao Prolog
  
    - many features aimed at teaching Prolog.

- These students typically have been exposed to other languages
    (imperative/OO, sometimes functional) and possibly logic,
    specifications, some notions of PL implementation, etc.

- Our related teaching materials -- slides, examples, Active Logic
  Documents (ALDs): <https://cliplab.org/logalg>
}


- Here: expand on one of the main messages: **do show the beauty!**

  - That to solve a problem, we can just describe it in logic and ask
    questions. 

  @p

  - That logic programs are both: 

    - **logical theories** / **specifications** (with declarative
      meaning) and

    - **procedural programs** that can be debugged, followed step by
      step, etc. 

    @p

    @comment{Show with examples (and benchmarking them) how}

  - That one can go from executable specifications to efficient
    algorithms gradually, % and 
    as needed.

@comment{
# A good start: explaining Green's proposal (I)

- A new view of computing:

@p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/green_lcolor-full}

# A good start: explaining Green's proposal (II)

- Prolog is the materialization of this dream::

@p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/green_lcolor-prolog}

  Robinson + Colmerauer + Kowalski / Kuhnen's SL-Resolution over Horn clauses 

→ Prolog (LP) 

→ Correct Answers / Results 

+ Warren 

→ with very competitive execution time!

# But then: 

- No correctness proofs needed?
- Even no programming needed?
- A specification and also a program?
- Is this possible?
}


# The vision is easy to convey at first 

- The first examples - family relations.

```ciao_runnable
:- module(_,_).

%! \begin{focus}
mother_of(susan, mary).
mother_of(susan, john).
mother_of(mary, michael).

father_of(john, david).

grandmother_of(X,Y) :-
    mother_of(X,Z), mother_of(Z,Y).
grandmother_of(X,Y) :-
    mother_of(X,Z), father_of(Z,Y).
%! \end{focus}
```

Students can get answers to questions in different modes:

```ciao_runnable
?- mother_of(susan,Y).
```
```ciao_runnable
?- mother_of(X,mary).
```
```ciao_runnable
?- grandmother_of(X,Y).
```

All is good!

# The vision is easy to convey at first 

- Some fun examples - circuit topology.

@p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/logic_circuit_color_small}


```ciao_runnable
:- module(_,_).

%! \begin{focus}
resistor(power,n1).
resistor(power,n2).

transistor(n2,ground,n1).
transistor(n3,n4,n2).
transistor(n5,ground,n4).

inverter(Input,Output) :- 
   transistor(Input,ground,Output), resistor(power,Output).

nand_gate(Input1,Input2,Output) :- 
   transistor(Input1,X,Output), transistor(Input2,ground,X), 
   resistor(power,Output).
   
and_gate(Input1,Input2,Output) :-
   nand_gate(Input1,Input2,X), inverter(X, Output).
%! \end{focus}
```

```ciao_runnable
?- and_gate(In1,In2,Out).
```

All still good!

**All these queries will return the correct solution and terminate using standard Prolog.**

However, things can quickly get more complicated...

# Non-termination rears its ugly head

@comment{A first source of non-termination}

```ciao_runnable
:- module(_,_).

%! \begin{focus}
mother_of(susan, mary).
mother_of(susan, john).
mother_of(mary, michael).

father_of(john, david).

parent(X,Y) :- mother_of(X,Y).
parent(X,Y) :- father_of(X,Y).

ancestor(X,Y) :- ancestor(X,Z), parent(Z,Y).
ancestor(X,Y) :- parent(X,Y).
%! \end{focus}
```

Now, for _any_ query, standard Prolog hangs:

```ciao_runnable
?- ancestor(X,Y).
```


# Non-termination rears its ugly head

```ciao_runnable
:- module(_,_).

%! \begin{focus}
mother_of(susan, mary).
mother_of(susan, john).
mother_of(mary, michael).

father_of(john, david).

parent(X,Y) :- mother_of(X,Y).
parent(X,Y) :- father_of(X,Y).

ancestor(X,Y) :- parent(X,Y).
ancestor(X,Y) :- ancestor(X,Z), parent(Z,Y).
%! \end{focus}
```

**Changing the order of the ancestor/2 clauses**:

```ciao_runnable
?- ancestor(X,Y).
```

Standard Prolog obtains all the answers and then hangs.

# Key point: it is a bit early to face non-termination!

- It is an **inevitable fact** that sooner or later students will have
  to deal with non-termination.

@p

- But having to learn how to get around it early on can lead to
  confusion / disenchantment.
  
@p

- Need to provide early on additional **tools** and **understanding**.

# Tabling to the (partial) rescue

- Pioneered by XSB and now in B-Prolog, Ciao, SWI, YAP.

- Ensures termination for programs with the **bounded term size
  property**: 
  
  All sizes of subgoals and answers < some fixed number.
  
  (This is the case for all ``Datalog'' programs.)

- Mark `ancestor/2`as a *tabled predicate*.

# Tabling to the (partial) rescue

- Pioneered by XSB and now in B-Prolog, Ciao, SWI, YAP.

- Ensures termination for programs with the **bounded term size
  property**: 
  
  All sizes of subgoals and answers < some fixed number.
  
  (This is the case for all ``Datalog'' programs.)

- Mark `ancestor/2`as a *tabled predicate*:

```ciao
:- table ancestor/2.
```


```ciao_runnable
?- ancestor(X,Y).
```

Now we not only obtain all answers (even with the first clause order):

```
X = susan,
Y = mary ? ;
X = susan,
Y = john ? ;
X = mary,
Y = michael ? ;
X = john,
Y = david ? ;
X = susan,
Y = michael ? ;
X = susan,
Y = david ? ;
no
```

but the program also terminates.

We can get this result also with standard Prolog, but need to
switch  order of goals in the body: 

```ciao_runnable
:- module(_,_).

%! \begin{focus}
mother_of(susan, mary).
mother_of(susan, john).
mother_of(mary, michael).

father_of(john, david).

parent(X,Y) :- mother_of(X,Y).
parent(X,Y) :- father_of(X,Y).

ancestor(X,Y) :- parent(X,Y).
ancestor(X,Y) :- parent(Z,Y), ancestor(X,Z).
%! \end{focus}
```

```ciao_runnable
?- ancestor(X,Y).
```


However, this is not always the case and tabling offers a more
powerful mechanism.

@p

@p

**Some relevant issues in tabling:**

- Which predicates to declare as tabled

  - cf. XSB's `:- auto_table.`

@p

- Does not cover cases outside the bounded term-size property.

@comment{
# More complex cases

```ciao_runnable
:- module(_,_).

%! \begin{focus}
natural(0).
natural(s(X)) :- natural(X).

pair(X,Y) :- natural(X), natural(Y).
%! \end{focus}
```

```ciao_runnable
?- pair(X,Y), X=s(0).
```

- Hangs in standard Prolog (never progresses beyond `X=0`, because of
  infinite solutions for `Y`).
  
- In this case the surprised student may realize it suffices to change
  the order: 

```ciao_runnable
?- X=s(0), pair(X,Y).
```

  and will see that now all answers are enumerated. 
}

# More complex cases

```ciao_runnable
:- module(_,_).

%! \begin{focus}
natural(0).
natural(s(X)) :- natural(X).

add(0,Y,Y) :- natural(Y).
add(s(X),Y,s(Z)) :- add(X,Y,Z).

mult(0,Y,0) :- natural(Y).
mult(s(X),Y,Z) :- add(W,Y,Z), mult(X,Y,W).

nat_square(X,Y) :- natural(X), natural(Y), mult(X,X,Y).
%! \end{focus}
```

- In class, we develop each of predicate with the students reasoning
  about the (infinite) set of (ground) facts we want to capture
  (declarative semantics).

# More complex cases

```ciao_runnable
:- module(_,_).

%! \begin{focus}
natural(0).
natural(s(X)) :- natural(X).

add(0,Y,Y) :- natural(Y).
add(s(X),Y,s(Z)) :- add(X,Y,Z).

mult(0,Y,0) :- natural(Y).
mult(s(X),Y,Z) :- add(W,Y,Z), mult(X,Y,W).

nat_square(X,Y) :- natural(X), natural(Y), mult(X,X,Y).
%! \end{focus}
```

Standard Prolog execution provides useful answers for, e.g.,
mode `nat_square(+,-)`.

```ciao_runnable
?- nat_square(s(s(0)), X).
```

and even for mode `nat_square(-,+)` which nicely surprises students: 


```ciao_runnable
?- nat_square(X,s(s(s(s(0))))).
```

- The next obvious temptation is to try:

```ciao_runnable
?- nat_square(X,Y).
```

- but only the first, trivial answer is generated and then execution hangs.

- Cannot be avoided with just tabling. 

# Fairness to the (partial) rescue

- To deal with these more complex cases we have found it useful to
  provide a means for selectively switching to other (fair) search
  rules such as breadth-first or iterative deepening: 
  
```ciao
:- use_package(sr/bfall).
```

- Comes with Ciao Prolog, but can be implemented in any Prolog with
  meta-programming. 
  
- Of course, with a fair rule **everything "works well"** for **all
  possible queries**.
  
  - In the sense that all valid solutions will be found, even if
    possibly inefficiently.

# Fairness to the (partial) rescue

For example, our problematic query will now enumerate the infinite set
of correct answers.

```ciao_runnable
:- module(_,_).

%! \begin{focus}
natural(0).
natural(s(X)) :- natural(X).

add(0,Y,Y) :- natural(Y).
add(s(X),Y,s(Z)) :- add(X,Y,Z).

mult(0,Y,0) :- natural(Y).
mult(s(X),Y,Z) :- add(W,Y,Z), mult(X,Y,W).

nat_square(X,Y) :- natural(X), natural(Y), mult(X,X,Y).
%! \end{focus}
```

```ciao_runnable
?- nat_square(X,Y).
```

# Other benefits of the fair search rules

- Even for the cases in which depth-first works, in bf:

  - The ``quality'' in the solution enumeration greatly improves!

- For example, for the Peano example, in depth-first search:

```ciao_runnable
:- module(_,_).

%! \begin{focus}


natural(0).
natural(s(X)) :- natural(X).

add(0,Y,Y) :- natural(Y).
add(s(X),Y,s(Z)) :- add(X,Y,Z).

mult(0,Y,0) :- natural(Y).
mult(s(X),Y,Z) :- add(W,Y,Z), mult(X,Y,W).
%! \end{focus}
```

```ciao_runnable
?- mult(X,Y,Z).
```

- It is clear that some answers will never be generated. 
- Would be a bad trainer for a learning system!

# Other benefits of the fair search rules

- Even for the cases in which depth-first works

  - the ``quality'' in the solution enumeration greatly improves!

- While in breadth-first search:

```ciao_runnable
:- module(_,_).

%! \begin{focus}
:- use_package(sr/bfall).

natural(0).
natural(s(X)) :- natural(X).

add(0,Y,Y) :- natural(Y).
add(s(X),Y,s(Z)) :- add(X,Y,Z).

mult(0,Y,0) :- natural(Y).
mult(s(X),Y,Z) :- add(W,Y,Z), mult(X,Y,W).
%! \end{focus}
```

```ciao_runnable
?- mult(X,Y,Z).
```
  
- Clearly more informative, satisfying, and useful for subsequent
  predicates.

@comment{
# 

@begin{alert}

In summary, we argue that the use of a fair search rule helps students
visualize the true potential of the (C)LP paradigm and Prolog -- that
it is possible indeed to solve problems by simply thinking logically
and/or inductively.

@end{alert}
}

# Other benefits of the fair search rules

- In Ciao Prolog marking predicates as types (or in general as
  properties) allows them to be used in assertions. 
  
- However, remain regular predicates can be:

  - called as any other, 
  - used as run-time tests (dynamic checking), 
  - "run backwards" to generate examples or test cases
    (property-based testing), etc.

# Other benefits of the fair search rules

- In Ciao Prolog marking predicates as types (or in general as
  properties) allows them to be used in assertions. 
  
- However, remain regular predicates can be:

  - called as any other, 
  - used as run-time tests (dynamic checking), 
  - **"run backwards" to generate examples or test cases**
    (property-based testing), etc.

For example: 

```ciao_runnable
:- module(_,_).

%! \begin{focus}
:- use_package([assertions,regtypes]).
% :- use_package(sr/bfall).
  
:- regtype color/1.
color(red).
color(green).
color(blue).

:- regtype colorlist/1.
colorlist([]).
colorlist([H|T]) :- color(H), colorlist(T).

%! \end{focus}
```

```ciao_runnable
?- colorlist(L).
```

- Depth-first correct answers, but not very satisfying.
- Breadth-first much better!


# Other benefits of the fair search rules

- In Ciao Prolog marking predicates as types (or in general as
  properties) allows them to be used in assertions. 
  
- However, remain regular predicates can be:

  - called as any other, 
  - used as run-time tests (dynamic checking), 
  - **"run backwards" to generate examples or test cases**
    (property-based testing), etc.

For example (in Ciao's *functional notation*): 

```ciao_runnable
:- module(_,_).

%! \begin{focus}
:- use_package([fsyntax,assertions,regtypes]).
% :- use_package(sr/bfall).
  
:- regtype color/1. 
color := red | green | blue.



:- regtype colorlist/1. 
colorlist := [] | [~color|~colorlist].

%! \end{focus}
```

```ciao_runnable
?- colorlist(L).
```

- Depth-first correct answers, but not very satisfying.
- Breadth-first much better!

# Fair search rules are great, but they cannot do the impossible

- After playing with a fair search rule, students will observe that it
  does provide all the expected answers, but sometimes hangs after
  that.
  
- Ah, that pesky semi-decidabilty (a.k.a, halting problem)... 

  - We cannot completely solve non-termination!

- It is important thus to explain better what is going on.

@comment{
# Explaining non-termination:

- Non-termination is a fact of life for any powerful programming
  language or proof system.

- However, it is likely to discourage beginners if not explained well:

  - Use/build system to run *alternatively and selectively* in
    breadth-first, iterative deepening, tabling, etc.

  - Start by running all predicates, e.g., breadth-first --
    everything works!

  - Then, explain the shape of the tree (solutions at finite depth,
    possible infinite failures, etc.), and thus why breadth-first
    works, and why depth-first sometimes may not.
}

# Characterization of the search tree

@p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/search_cases_small}

- All solutions are at *finite depth* in the tree.
- Failures can be at finite depth or, in some cases, be an infinite branch.

# Depth-First Search

@p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/search_cases_df_small}

- Incomplete: may fall through an infinite branch before finding all solutions.
- But very efficient: it can be implemented with a call stack, very similar to a traditional programming language.



# Breadth-First Search

@p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/search_cases_bf_small} 

- Will find all solutions before falling through an infinite branch.
- But costly in terms of time and memory.
- Used in all the following examples (via Ciao's `bf` package).



@comment{
# Explaining non-termination:

- Relate to/mention semi-decidability and the *halting problem*: no-one
  (Prolog, logic, nor other Turing-complete prog. language) can
  solve that (but tabling helps: good time to introduce it!).

- Discuss advantages and disadvantages of search rules (time,
  memory).

- Motivate the choices made for Prolog benchmarking actual
  executions.
}

# From specifications to efficient programs

- OK, so can we have executable specifications. 

  - Are they efficient?

    - Sometimes, efficiency not even needed...

  - If not, do we then need to go to an imperative language?
  
    - No, we can gradually optimize the performance-sensitive parts
      within Prolog


# From Specifications to Efficient Programs

- Defining `mod(X,Y,Z)`
  @p ''`Z` is the remainder from dividing `X` by `Y`''
  @p @math{@exists Q s.t. X = Y * Q + Z @wedge Z < Y} 
  @comment{ mod(X,Y,Z) ⇔ ∃Qs.t. X = Y ∗ Q + Z ∧ Z < Y }
  @p @math{@Rightarrow}
- Can be expressed directly in Prolog:

```ciao_runnable
:- module(_,_).

%! \begin{focus}
:- use_package(sr/bfall).
mod(X,Y,Z) :- less(Z, Y), mult(Y,_Q,W), add(W,Z,X).
%! \end{focus}
```

- Program clearly correct (it is the specification!).
- And with bf all the infinite solutions are enumerated.

```ciao_runnable
?- op(500,fy,s).
?- mod(X,Y, s 0).
```

- But it can of course be quite inefficient.

- However, we can write a much more efficient version, also within
  (pure) Prolog:

  - Works well in depth-first for several modes.
  - However, enumeration of solutions is again biased.

- Again, we can also show the constraints version.

- And we can discuss modes and how they affect determinism, cost,
  termination, etc.

@comment{
# Clarifying the point 

- Not meant to say that df or even purity are necessary to go from
  specification to efficient program.
  
```ciao_runnable
:- module(_,_).

%! \begin{focus}
max(L,Max) :-
    member(Max,L),
    \+ (member(E,L), E>Max).


max2([H|T],Max) :-
    max_(T,H,Max).

max_([],Max,Max).
max_([H|T],TMax,Max) :-
       H > TMax,
       max_(T,H,Max).
max_([H|T],TMax,Max) :-
       H =< TMax,
       max_(T,TMax,Max).
%! \end{focus}

nums_list(N,N,[]).
nums_list(X,N,[X|T]) :-
    X < N,
    X1 is X+1,
    nums_list(X1,N,T).

```

- Both can only be used in mode `max(+,-).`

- However, still a nice example of go from specification to efficient program.

```ciao_runnable
?- nums_list(0,10000,_L), time( max(_L,Max) ).
```

```ciao_runnable
?- nums_list(0,10000,_L), time( max2(_L,Max) ).
```
}


# Depth-first / Breadth-first / Iterative Deepening

@p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/data-id-1500-1500-10000-to-include}

- Just the Ciao naive implementations: no claim of optimality!
- Only one program, no claim of generality!

# Some comments on Peano arithmetic

- Useful as instructive, reversible substitute for non-reversible
  `is/2`, `>`, etc. for first parts of the course.

  - We also use constraints as alternative [as, e.g., Neumerkel & Triska]

    - But it requires explaining/justifying an extra, complex "black box": the solver.
    - Peano nicely only uses unification, which students need to be
      introduced to anyway.
    - We do mention that constraints are more efficient, but will use them more later.

- Peano also allows many elegant examples for the early steps of
  learning recursion and recursive data structures. 

# Final remarks

- Conveying the beauty.
- Having to deal with Prolog termination early on can work against.
- How to deal with it all within a Prolog system?
- Fair search rules and tabling come to the rescue. 
- Also constraints are very useful of course.

- Expanded here on one of many topics; see Prolog50 papers for more. 
- Se also: 
  - Our teaching materials: [https://cliplab.org/logalg](https://cliplab.org/logalg)
  - Other Ciao Prolog features for teaching (e.g., the `pure` package)
  - The Prolog Playground: [https://ciao-lang.org/playground](https://cliplab.org/logalg)
  - Active Logic Documents (e.g., this one!)
  - etc. 


# 

```ciao_runnable
%! \begin{jseval}
async (pg) => {
  let e = pg.preview_el;
  e.innerHTML = `
<h1>Thanks!</h1>
 `;
  pg.show_preview(true);
  pg.update_inner_layout();
}
%! \end{jseval}
```







# Polynomials

- Recognizing (and generating!) polynomials in some term X:
  - X is a polynomial in X
  - a constant is a polynomial in X
  - sums, differences and products of polynomials in X are polynomials
  - also polynomials raised to the power of a natural number and the quotient of a polynomial by a constant
    
    ```ciao
    polynomial(X,X).
    polynomial(Term,X)        :- pconstant(Term).
    polynomial(Term1+Term2,X) :- polynomial(Term1,X), polynomial(Term2,X). 
    polynomial(Term1-Term2,X) :- polynomial(Term1,X), polynomial(Term2,X).
    polynomial(Term1*Term2,X) :- polynomial(Term1,X), polynomial(Term2,X).
    polynomial(Term1/Term2,X) :- polynomial(Term1,X), pconstant(Term2).
    polynomial(Term1^N,X)     :- polynomial(Term1,X), nat(N).
    ```
    

# Derivative (and Integrals!)

- `deriv(Expression, X, Derivative)`
  
  ```ciao
  deriv(X,X,s(0)).
  deriv(C,X,0)                       :- pconstant(C).
  deriv(U+V,X,DU+DV)                 :- deriv(U,X,DU), deriv(V,X,DV). 
  deriv(U-V,X,DU-DV)                 :- deriv(U,X,DU), deriv(V,X,DV).
  deriv(U*V,X,DU*V+U*DV)             :- deriv(U,X,DU), deriv(V,X,DV).
  deriv(U/V,X,(DU*V-U*DV)/V^s(s(0))) :- deriv(U,X,DU), deriv(V,X,DV).
  deriv(U^s(N),X,s(N)*U^N*DU)        :- deriv(U,X,DU), nat(N).
  deriv(log(U),X,DU/U)               :- deriv(U,X,DU).
  ...
  ``` 
  
- `?- deriv(s(s(s(0)))*x+s(s(0)),x,Y).`

- A simplification step can be added.

# Recursive Programming: Automata (Graphs)


- Recognizing the sequence of characters accepted by the following *non-deterministic, finite automaton* (NDFA):
  @p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/autom1_color}
  where **q0** is both the *initial* and the *final* state.


- Strings are represented as lists of constants (e.g., `[a,b,b]`).

- Program:
  ```ciao
  initial(q0).      delta(q0,a,q1).
                    delta(q1,b,q0).
  final(q0).        delta(q1,b,q1).
  
  accept(S)        :- initial(Q), accept_from(S,Q).
  
  accept_from([],Q)     :- final(Q).
  accept_from([X|Xs],Q) :- delta(Q,X,NewQ), accept_from(Xs,NewQ).
  ```


# Recursive Programming: Automata (Graphs) (Contd.)

- A *nondeterministic, stack, finite automaton* (NDSFA):
  ```ciao
  accept(S) :- initial(Q), accept_from(S,Q,[]).
  
  accept_from([],Q,[])    :- final(Q).
  accept_from([X|Xs],Q,S) :- delta(Q,X,S,NewQ,NewS), 
                accept_from(Xs,NewQ,NewS).
  
  initial(q0).
  final(q1).
  
  delta(q0,X,Xs,q0,[X|Xs]).
  delta(q0,X,Xs,q1,[X|Xs]).
  delta(q0,X,Xs,q1,Xs).
  delta(q1,X,[X|Xs],q1,Xs).
  ```
  
- What sequence does it recognize?



# Recursive Programming: Towers of Hanoi


- *Objective:*
  - Move tower of N disks from peg `a` to peg `b`, with the help of peg `c`. 

- *Rules:*
  - Only one disk can be moved at a time.
  - A larger disk can never be placed on top of a smaller disk.

  @p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/hanoi}{1000}{325}

# Recursive Programming: Towers of Hanoi (Contd.)


- We will call the main predicate `hanoi_moves(N,Moves)`

- `N` is the number of disks and `Moves` the corresponding list of ''moves''. 

- Each move `move(A, B)` represents that the top disk in A should be moved to B.

- *Example:*
  @p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/hanoi3}{1000}{115}

  @p is represented by: 
  ```ciao
  hanoi_moves( s(s(s(0))), 
               [ move(a,b), move(a,c), move(b,c), move(a,b),
                 move(c,a), move(c,b), move(a,b) ])
  ```

# Recursive Programming: Towers of Hanoi (Contd.)

- A general rule:
  @p @image{https://github.com/mherme/ald-tests/edit/main/teachingws/Figs/hanoi_gen}{1000}{65}

- We capture this in a predicate `hanoi(N,Orig,Dest,Help,Moves)` where
  @p ''`Moves` contains the moves needed to move a tower of `N`disks from peg `Orig` to peg `Dest`, with the help of peg `Help`.''
  ```ciao
  hanoi(s(0),Orig,Dest,_Help,[move(Orig, Dest)]).
  hanoi(s(N),Orig,Dest,Help,Moves) :- 
          hanoi(N,Orig,Help,Dest,Moves1),
          hanoi(N,Help,Dest,Orig,Moves2), 
          append(Moves1,[move(Orig, Dest)|Moves2],Moves).
  ```

- And we simply call this predicate:
  ```ciao 
  hanoi_moves(N,Moves) :- 
  hanoi(N,a,b,c,Moves).
  ```

# On system types

- System types: 

    -   Classical installation: most appropriate for more advanced
        students / "real" use.
        Show serious, competitive language.

    -   Playgrounds and notebooks --see PL50 Book papers!
        (e.g., Ciao Playground/Active Logic Documents, SWISH,
        $\tau$-Prolog, SICStus+Jupyter).

        -   Can be attractive for beginners, young students.

        -   Some places (e.g., schools) may not have personnel/machines
            for installation, but will have a tablet.

        -   Server-based vs. browser-based.

        -   Very useful for executable examples in manuals and
            tutorials.

    Offer both types to students!

-   Block-based versions can be useful starters for youngest (cf. Laura
    Cecchi's paper in PL50 Book)

    -   Also, nice tool for kids developed by J.F.Morales and S. Abreu
        for the Prolog Year (see PL50 Book).

-   Ideally the system should allow covering:

    -   pure LP (with several search rules, tabling),

    -   ISO-Prolog,

    -   higher-order support and functional syntax,

    -   constraints,

    -   ASP/s(CASP), etc.

Much more in the PL50 book papers and teaching materials in
[https://cliplab.org/logalg](https://cliplab.org/logalg)


# Recursive Programming: Arithmetic/Functions

- The Ackermann function:
  ```ciao
  ackermann(0,N) = N+1
  ackermann(M,0) = ackermann(M-1,1)
  ackermann(M,N) = ackermann(M-1,ackermann(M,N-1))
  ```

- In Peano arithmetic:
  ```ciao
  ackermann(0,N)         = s(N)
  ackermann(s(M1),0)     = ackermann(M1,s(0))
  ackermann(s(M1),s(N1)) = ackermann(M1,ackermann(s(M1),N1))
  ```

- Can be defined as:
  ```ciao
  ackermann(0,N,s(N)).
  ackermann(s(M1),0,Val)     :-  ackermann(M1,s(0),Val).
  ackermann(s(M1),s(N1),Val) :-  ackermann(s(M1),N1,Val1),
                                 ackermann(M1,Val1,Val).
  ```

- In general, *functions* can be coded as a predicate with one more argument, which represents the output (and additional syntactic sugar often available).


# Recursive Programming: Arithmetic/Functions (Functional Syntax)


- Syntactic support available (see, e.g., the Ciao *fsyntax* and *functional* packages).

- The Ackermann function (Peano) in Ciao's functional Syntax and defining `s` as a prefix operator:
  ```ciao
  :- use_package(functional).
  :- op(500,fy,s).
  
  ackermann(  0,   N) := s N.
  ackermann(s M,   0) := ackermann(M, s 0).
  ackermann(s M, s N) := ackermann(M, ackermann(s M, N) ).
  ```

- Convenient in other cases -- e.g. for defining types: 
  ```ciao
  nat(0).
  nat(s(X)) :- nat(X).
  ```

  Using special `:=` notation for the ''return'' (last) the argument: 
  ```ciao
  nat := 0.
  nat := s(X) :- nat(X).
  ```

# Recursive Programming: Arithmetic/Functions (Funct. Syntax, Contd.)

Moving body call to head using the `~` notation (''evaluate and replace with result''): 
```ciao
nat := 0.
nat := s(~nat).
```

''`~`'' not needed with `funcional` package if inside its own definition: 
```ciao
nat := 0.
nat := s(nat).
```

Using an `:- op(500,fy,s).` declaration to define `s` as a *prefix operator*:
```ciao
nat := 0.
nat := s nat.
```

Using ''`|`'' (disjunction):
```ciao
nat := 0 | s nat.
```

Which is exactly equivalent to:
```ciao
nat(0).
nat(s(X)) :- nat(X).
```
