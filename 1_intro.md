\title A Motivational Introduction

\begin{note}
This is an interactive HTML version of the [**Course on logic and constraint logic programming**](http://www.cliplab.org/prode/): [1 - Motivation](http://www.cliplab.org/logalg/slides/1_intro.pdf)
\end{note}

# The Program Correctness Problem

@image{https://github.com/mherme/ald-tests/edit/main/Figs/logic_use_1.png} 

 - Conventional models of using computers -- not easy to determine
   correctness!

   - Has become a very important issue, not just in safety-critical
     apps.

   - Components with assured quality, being able to give a warranty, ...

   - Being able to run untrusted code, certificate carrying code, ...

# A Simple Imperative Program

- Example:
```c
#include <stdio.h>

main() {
  int Number, Square;
  Number = 0;
  while (Number <= 5) {
    Square = Number * Number;
    printf('%d\\n',Square);
    Number = Number + 1;
  }
}
```
- Is it correct? With respect to what?
- A suitable formalism is needed:
  - to provide *specifications* (describe problems), and
  - to reason about the *correctness of programs* (their *implementation*).

# Natural Language

*''Compute the squares of the natural numbers which are less or equal than 5.''*

Ideal at first sight, but:
 - verbose
 - vague
 - ambiguous
 - needs context (assumed information)
 - ...

Philosophers and mathematicians already pointed this out a long time ago... 

# Logic

- A means of clarifying / formalizing the human thought process

- Logic for example tells us that (classical logic)
  
  *Aristotle likes cookies, and*

  *Plato is a friend of anyone who likes cookies*

  **imply that**
    
  *Plato is a friend of Aristotle*

- Symbolic logic: A shorthand for classical logic -- plus many useful
  results:

  @math{a_1: likes(aristotle,cookies)}

  @math{a_2: @forall X likes(X,cookies) @rightarrow friend(plato,X)}

  @math{t_1: friend(plato,aristotle)}

  @math{T[a_1,a_2] @vdash t_1}

- But, can logic be used:

  - To represent the problem (specifications)?
  - *Even perhaps to solve the problem?*

# Using Logic

@image{logic_use_2a} 

- For expressing specifications and reasoning about the
  correctness of programs we need:

   - Specification languages (assertions), modeling, ... 
   - Program semantics (models, axiomatic, fixpoint, ...).
   - Proofs: program *verification* (and debugging, equivalence, ...).

# Generating Squares: A Specification (I)

Numbers ---we will use *''Peano''* representation for simplicity:
@p  @math{0 @equiv 0 @ @ @ 1 @equiv s(0) @ @ @ 2 @equiv s(s(0)) @ @ @ 3 @equiv s(s(s(0))) @ @ @ @dots }

 - Defining the natural numbers:
   @p 0 is a natural, @hfill 1 is a natural, @hfill 2 is a natural,@hfill @math{@dots}
   @p @math{nat(0) @ @ @ @ @   @wedge @ @ @ @ @ nat(s(0))  @ @ @ @ @   @wedge @ @ @ @ @ nat(s(s(0)))  @ @ @  @wedge  @dots}

   - A better solution:
     @begin{cartouche}
     @math{nat(0)@ @ @ @wedge @ @ @ @forall X @ (nat(X) @rightarrow nat(s(X)))}
     @end{cartouche}

 - Order on the naturals (less or equal than):
   @p @math{le(0,0) @ @ @ @ @ le(0,s(0)) @ @ @ @ @ le(0,s(s(0))) @ @ @ @ @ @dots }
   @p @math{le(s(0),s(0)) @ @ @ @ @ le(s(0),s(s(0))) @ @ @ @ @ le(s(0),s(s(s(0)))) @ @ @ @ @ @ldots}
   @begin{cartouche}
   @math{@forall X @ (nat(X) @rightarrow le(0,@ X)) @ @ @ @ @ @wedge @ @ @ @ @forall X @forall Y @ (le(X,Y) @rightarrow le(s(X),s(Y)))}
   @end{cartouche}

 - Addition of naturals:
   @p @math{ add(0,0,0) @ @ @ @ add(0,s(0),s(0)) @ @ @ @ add(0,s(s(0)),s(s(0))) @ @ @ @ @ldots  }
   @p @math{add(s(0),0,s(0)) @ @ @ @ add(s(0),s(0),s(s(0))) @ @ @ @ add(s(0),s(s(0)),s(s(s(0)))) @ @ @ @ @ldots }
   @begin{cartouche}
   @math{@forall X @ (nat(X) @rightarrow add(0, X, X)) @ @ @  @wedge @ @ @ @forall X @forall Y @forall Z @ (add(X,Y,Z) @rightarrow add(s(X),Y,s(Z)))}
   @end{cartouche}

# Generating Squares: A Specification (II)

- Multiplication of naturals:
  *''Multiply @math{X} and @math{Y}''* is
  *''add @math{Y} to itself @math{X} times,''* e.g. 

  @p @math{mult(3,2,6) @equiv} adding 2 to itself 3 times is 6 @math{@equiv  2+2+2 = 6}
  @p @math{mult(3,2,6) @wedge add(6,2,8) @rightarrow mult(4,2,8) @ @ @     2+2+2+3 = 8}

  @begin{cartouche}
  @math{ @forall X @ (nat(X) @rightarrow mult(0, X, 0)) @ @wedge }
  @p @math{ @forall X @forall Y @forall Z @forall W @ (mult(X,Y,W) @wedge add(W,Y,Z) @rightarrow mult(s(X),Y,Z))}
  @end{cartouche}

- Squares of the naturals:
  @begin{cartouche}
  @math{@forall X @forall Y @ (nat(X) @wedge nat(Y) @ @wedge  mult(X,X,Y) @rightarrow nat@_square(X,Y))}
  @end{cartouche}

We can now write a *specification* of the (imperative) program, i.e.,
conditions that we want the program to meet:

- *Precondition (empty):*
  @begin{cartouche}
  @math{true}
  @end{cartouche}
- *Postcondition:*
  @begin{cartouche}
  @math{@forall X (output(X) @leftarrow (@exists Y @ nat(Y) @wedge le(Y,s(s(s(s(s(0)))))) @wedge nat@_square(Y,X)))}
  @end{cartouche}

# Alternative Use of Logic?

@begin{itemize}
@item So, logic allows us to @em{represent problems} (program specifications).

@p @image{logic_use_3}

@p @noindent i.e., the process of implementing solutions to problems.

@item The importance of Programming Languages (and tools).
@item Interesting question: can logic help here too?
@end{itemize}

# From Representation/Specification to Computation

@begin{itemize}

@item Assuming the existence of 
a @em{mechanical proof method} (deduction procedure)
@p @em{ a new view of problem solving and computing is possible} [Green]:
@begin{itemize}
@item program once and for all the deduction procedure in the computer,
@item find a suitable @em{representation} for the problem (i.e., the
@em{specification}), 
@item then, to obtain solutions, ask questions and let 
deduction procedure do rest:
@end{itemize} 
@image{greene_lcolor}
@item No correctness proofs needed!
@end{itemize}

# Computing With Our Previous Description / Specification

@math{
\def\query#1{{#1 \mathbb{?}}}
\def\answer#1{{#1}}
}
@math{
\begin{array}{ll}
  \textbf{Query} & \textbf{Answer} \\ 
\hline

\query{nat(s(0))} &
\answer{(yes)} \\
\hline

\query{\exists X \ add(s(0),s(s(0)),X)} &
\answer{X = s(s(s(0)))} \\
\hline

\query{\exists X \ add(s(0),X,s(s(s(0))))} &
\answer{X = s(s(0))} \\
\hline

\query{\exists X \ nat(X)} &
\answer{X = 0 \vee X = s(0) \vee X = s(s(0)) \vee \ldots} \\
\hline

\query{\exists X \exists Y \ add(X,Y,s(0))} &
\answer{(X = 0 \wedge Y=s(0)) \vee (X = s(0) \wedge Y = 0)} \\
\hline

\query{\exists X \ nat\_square(s(s(0)), X)} &
\answer{X = s(s(s(s(0))))} \\
\hline

\query{\exists X \ nat\_square(X,s(s(s(s(0)))))} &
\answer{X = s(s(0))} \\
\hline

\query{\exists X \exists Y \ nat\_square(X,Y)} &
\answer{ (X = 0 \wedge Y=0) \vee 
          (X = s(0) \wedge Y=s(0)) \vee 
          (X = s(s(0)) \wedge Y=s(s(s(s(0))))) \vee 
          \ldots } \\
\hline

\query{\exists X output(X)} &
\answer{ X = 0 \ \vee \  
         X = s(0) \ \vee \  
         X = s(s(s(s(0)))) \ \vee \ 
         X = s^9(0) \ \vee \ 
         X = s^{16}(0) \ \vee \ 
         X = s^{25}(0)
} \\
\hline
\end{array}
}

# Which Logic?

@begin{itemize} 

@item We have already argued the convenience of representing the
problem in logic, but

@begin{itemize}
@item which logic?
@begin{itemize}
@item propositional
@item predicate calculus (first order)
@item higher-order logics
@item modal logics
@item @math{@lambda}-calculus
@item ...
@end{itemize}
@item which reasoning procedure?
@begin{itemize}
@item natural deduction, classical methods
@item resolution
@item Prawitz/Bibel, tableaux
@item bottom-up fixpoint
@item rewriting
@item narrowing
@item ...
@end{itemize}
@end{itemize}
@end{itemize}

# Issues

@begin{itemize}
@item We try to @bf{maximize expressive power}.
Example: propositions vs. first-order formulas.
@begin{itemize}
@item @em{Propositional} logic:
@begin{itemize}
@p ''spot is a dog'' @math{p}
@p ''dogs have tail'' @math{q}
@end{itemize}
But how can we conclude that Spot has a tail?

@item @em{Predicate} logic extends the @em{expressive power} of propositional logic:
@begin{itemize}
@p @math{dog(spot)}
@p @math{@forall X dog(X) @rightarrow has@_tail(X)}
@end{itemize}
Now, using deduction we can conclude:
@begin{itemize}
@p @math{has@_tail(spot)}
@end{itemize}
@end{itemize}

@item But one of the main issues is whether we have an
@bf{effective reasoning procedure}.

@p @math{@rightarrow} It is important to understand the underlying properties and the
theoretical limits!
@end{itemize}


# Comparison of Logics (I)

@begin{itemize}
@item @bf{Propositional} logic:

@begin{itemize}
''spot is a dog'' @em{p}
@p + decidability
@p - limited expressive power
@p + practical deduction mechanism 
@end{itemize}
@math{@rightarrow} Circuit design, ''answer set'' programming, ...


@item @bf{Predicate logic}: (first order)

@begin{itemize}
''spot is a dog'' @em{dog(spot)}
@p +/- decidability
@p +/- good expressive power
@p + practical deduction mechanism (e.g., @bf{SLD-resolution})
@end{itemize}
@math{@rightarrow} Classical logic programming!
@end{itemize}


# Comparison of Logics (II)

@begin{itemize}
@item @bf{Higher-order predicate logic}: 
@begin{itemize}
@p ''There is a relationship for spot'' @em{X(spot)}
@p - decidability
@p + good expressive power
@p -- practical deduction mechanism
@end{itemize}
But interesting subsets @math{@rightarrow} HO logic programming, functional-logic
prog., ...
@item @bf{Other logics}: Decidability? Expressive power? Practical
deduction mechanism?
@p Often (very useful) variants of previous ones:
@begin{itemize}
@item Predicate logic + constraints (in place of unification)
@p @math{@rightarrow} constraint programming!
@item Propositional temporal logic, etc.
@end{itemize}
@item Interesting case: @bf{@math{@lambda}-calculus}
@begin{itemize}
@p + similar to predicate logic in results, allows higher order
@p - does not support predicates (relations), only functions
@end{itemize}
@math{@rightarrow} Functional programming!
@end{itemize}

# Generating squares by SLD-Resolution -- Logic Programming (I)

@begin{itemize}
@item We code the problem as definite (Horn) clauses:
@begin{itemize}
@p @math{nat(0)}
@p @math{@neg nat(X) @vee nat(s(X))}
@p @math{@neg nat(X) @vee add(0, X, X))}
@p @math{@neg add(X,Y,Z) @vee add(s(X),Y,s(Z))}
@p @math{@neg nat(X) @vee mult(0, X, 0)}
@p @math{@neg mult(X,Y,W) @vee @neg add(W,Y,Z) @vee mult(s(X),Y,Z)} 
@p @math{@neg nat(X) @vee @neg nat(Y) @vee @neg mult(X,X,Y) @vee nat@_square(X,Y)}
@end{itemize}

@item @bf{Query:}  @math{nat(s(0))}  @bf{?}

@begin{itemize}
@item In order to refute: @math{@neg nat(s(0))}
@item Resolution:
@p @math{@neg nat(s(0))} and @math{@neg nat(X) @vee nat(s(X))} with unifier @math{@{X/0@}} giving @math{@neg nat(0)}
@p @math{@neg nat(0)} and @math{nat(0)} with unifier @math{@{@}}  giving @math{@Box}
@item Answer: @math{(yes)}       

@end{itemize}
@end{itemize}

# Generating squares by SLD-Resolution -- Logic Programming (II)

@begin{itemize}
@item We code the problem as definite (Horn) clauses:
@begin{itemize}
@p @math{nat(0)}
@p @math{@neg nat(X) @vee nat(s(X))}
@p @math{@neg nat(X) @vee add(0, X, X))}
@p @math{@neg add(X,Y,Z) @vee add(s(X),Y,s(Z))}
@p @math{@neg nat(X) @vee mult(0, X, 0)}
@p @math{@neg mult(X,Y,W) @vee @neg add(W,Y,Z) @vee mult(s(X),Y,Z)} 
@p @math{@neg nat(X) @vee @neg nat(Y) @vee @neg mult(X,X,Y) @vee nat@_square(X,Y)}
@end{itemize}

@item @bf{Query:} @math{@exists X @exists Y @ add(X,Y,s(0))} @bf{?}

@begin{itemize}
@item In order to refute: @math{@neg add(X,Y,s(0))}
@item Resolution:
@p @math{@neg add(X,Y,s(0))} and @math{@neg nat(X) @vee add(0, X, X))} with unifier @math{@{X=0, Y=s(0)@}} 
@p giving  @math{@neg nat(s(0))} (and @math{@neg nat(s(0))} is resolved as before)
@item Answer: @math{X=0, Y=s(0)}
@item Alternative:
@p @math{@neg add(X,Y,s(0))} with @math{@neg add(X,Y,Z) @vee add(s(X),Y,s(Z))}
    giving @math{@neg add(X,Y,0)} @math{@dots}
@end{itemize}

@end{itemize}


# Generating Squares in a Practical Logic Programming System (I)

```ciao_runnable
:- module(_, _, [assertions]).

:- test add(A, B, C) : (A = 0, B = 0)
   => (C = 0) + (not_fails, is_det).
:- test add(A, B, C) : (B = 0, C = 0)
   => (A = 0) + (not_fails, is_det).
:- test add(A, B, C) : (A = s(0), B = 0)
   => (C = s(0)) + (not_fails, is_det).
:- test add(A, B, C) : (A = s(0), B = s(s(0)))
   => (C = s(s(s(0)))) + (not_fails, is_det).
:- test add(A, B, C) : (B = s(0), C = s(s(0)))
   => (A = s(0)) + (not_fails, is_det).
:- test add(A, B, C) : (A = s(s(s(0))), B = s(s(0)))
   => (C = s(s(s(s(s(0)))))) + (not_fails, is_det).

%! \begin{hint}
nat(0).
nat(s(X)) :- nat(X).

le(0,X) :- nat(X).
le(s(X),s(Y)) :- le(X,Y).

add(0,Y,Y). %TODO

mult(0,Y,0) :- nat(Y).
mult(s(X),Y,Z) :- add(W,Y,Z), mult(X,Y,W).

nat_square(X,Y) :- nat(X), nat(Y), mult(X,X,Y).
output(X) :- nat(Y), le(Y,s(s(s(s(s(0)))))), nat_square(Y,X).
%! \end{hint}
%! \begin{solution}
nat(0).
nat(s(X)) :- nat(X).

le(0,X) :- nat(X).
le(s(X),s(Y)) :- le(X,Y).

add(0,Y,Y) :- nat(Y).
add(s(X),Y,s(Z)) :- add(X,Y,Z).

mult(0,Y,0) :- nat(Y).
mult(s(X),Y,Z) :- add(W,Y,Z), mult(X,Y,W).

nat_square(X,Y) :- nat(X), nat(Y), mult(X,X,Y).
output(X) :- nat(Y), le(Y,s(s(s(s(s(0)))))), nat_square(Y,X).
%! \end{solution}
```


# Generating Squares in a Practical Logic Programming System (II)

```
Query                              Answer
---------------------------------------------------------------------------
?- nat(s(0)).                      yes

?- add(s(0),s(s(0)),X).            X=s(s(s(0)))

?- add(s(0),X,s(s(s(0)))).         X=s(s(0))

?- nat(X).                         X=0 ; X=s(0) ; X=s(s(0)) ; ...

?- add(X,Y,s(0)).                  X=0, Y=s(0) ; X=s(0), Y=0

?- nat_square(s(s(0)), X).         X=s(s(s(s(0))))

?- nat_square(X,s(s(s(s(0))))).    X=s(s(0))

?- nat_square(X,Y).                X=0, Y=0 ; 
                                   X=s(0), Y=s(0) ; 
                                   X=s(s(0)), Y=s(s(s(s(0)))) ; 
                                   ...

?- output(X).                      X=0 ; X=s(0) ; X=s(s(s(s(0)))) ; ... 
```

# Additional examples (I) -- Family relations

The following logic formulas:
@p @math{father\_of(john, peter)}
@p @math{father\_of(john, mary)}
@p @math{father\_of(peter, michael)}
@p @math{mother\_of(mary, david)}
@p @math{@forall X@forall Y(@exists Z (father\_of(X,Z) @wedge father\_of(Z,Y)) @rightarrow grandfather\_of(X,Y))}
@p @math{@forall X@forall Y(@exists Z (father\_of(X,Z) @wedge mother\_of(Z,Y)) @rightarrow grandfather\_of(X,Y))}

@comment{unicode version? `∀ X ∀ Y (∃ Z (father_of(X,Z) ∧ father_of(Z,Y) → grandfather_of(X,Y))`}

Correspond to the Prolog program:
```ciao_runnable
:- module(_,_).

%! \begin{focus}
father_of(john, peter).
father_of(john, mary).
father_of(peter, michael).
mother_of(mary, david).

grandfather_of(L,M) :- father_of(L,K),
                       father_of(K,M).

grandfather_of(X,Y) :- father_of(X,Z),
                       mother_of(Z,Y).
%! \end{focus}
```

@image{family_1} 

 - How can `grandmother_of/2` be represented?
 - What does `grandfather_of(X,david)` mean? And `grandfather_of(john,X)`

# Additional examples (II) - Testing membership in lists

Declarative view:

 - Suppose there is a functor @math{f/2} such that @math{f(H, T)} represents a
   list with head @math{H} and tail @math{T}.

 - Membership definition: 
   @math{X @in L @leftrightarrow @left@{ @begin{array}{ll} X @mbox{ is the head of L} \\
                         @mbox{or} @ X @ @mbox{is member of the tail of L}
                       @end{array} @right.}
 - Using logic formulas:
   @p @math{@forall X@forall L(@exists T @ (L = f(X, T) @rightarrow member(X, L)))}
   @p @math{@forall X@forall L(@exists Z @exists T @ (L = f(Z, T) @wedge member(X,T) @rightarrow member(X,L)))}

 - Using Prolog:

```ciao
member(X, f(X, T)).
member(X, f(Z, T)) :- member(X,T).
```

Procedural view (but for checking membership only!):

 - Traverse the list comparing each element until @math{X} is found or
   list is finished:

```c
/* Testing array membership in C */
int member(int x,int list[LISTSIZE]) {
   for (int i = 0; i < LISTSIZE; i++)
     if (x == list[i]) return TRUE;
   return FALSE;
}
```

# A (very brief) History of Logic Programming (I)

 - **60's**
   - Green: programming as problem solving.
   - Green: @bf{resolution}.

 - **70's**
   - Colmerauer: specialized theorem prover (Fortran) embedding the
     procedural interpretation: First @bf{Prolog} (''Programmation et
     Logique'') interpreter.

   - Kowalski: procedural interpretation of Horn clause logic. Read:
     @p @math{A} if @math{B_1} and @math{B_2} and @math{@cdots} and
     @math{B_n} as: @p to solve (execute) @math{A}, solve (execute)
     @math{B_1} and @math{B_2} and,..., @math{B_n}

     @p Algorithm = logic + control.

   - D.H.D. Warren develops first @bf{compiler}, DEC-10 Prolog, almost
     completely written in Prolog.  Very efficient (same as
     Lisp). Top-level, debugger, very useful builtins, ... becomes the
     standard.

# A (very brief) History of Logic Programming (II)

 - **80's, 90's**
   - Major research in the basic paradigms and advanced implementation
     techniques: Japan (Fifth Generation Project), US (MCC), Europe
     (ECRC, ESPRIT projects), leading to the current EU ''framework
     research programs''.
   - Numerous commercial Prolog implementations, programming books,
     using the @em{ de facto } standard, the Edinburgh Prolog family.
   - Leading in @bf{1995} to The ISO Prolog standard.
   - Parallel and concurrent logic programming systems.
   - Constraint Logic Programming (@bf{CLP}): A major extension --
     opened new areas and even communities:
     - Commercial CLP systems with fielded applications.
     - Concurrent constraint programming systems.

 - **2000-...**
   - Many other extensions: full higher order, support for
     types/modes, concurrency, distribution, objects, functional
     syntax, ...
   - Highly optimizing compilers, automatic, automatic parallelism,
     automatic verification and debugging, advanced environments.

   Also, Datalog, Answer Set Programming (ASP) -- support for negation
   through stable models.

# A (very brief) History of Logic Programming (III)

 - Many applications:
   - Natural language processing
   - Scheduling/Optimization problems
   - Heterogeneous data integration
   - Program analyzers and verifiers
   - ...

   Many in combination with other languages. 

- Some examples:
   - The [IBM Watson System
     (2011)](https://en.wikipedia.org/wiki/Watson_(computer)) has
     [important parts written in
     Prolog](https://www.cs.nmsu.edu/ALP/2011/03/natural-language-processing-with-prolog-in-the-ibm-watson-system/).
   - [Clarissa](https://ti.arc.nasa.gov/tech/cas/user-centered-technologies/clarissa/),
     a voice user interface by NASA for browsing ISS procedures.
   - The first Erlang interpreter was developed in Prolog by Joe Armstrong.
   - The Microsoft Windows NT Networking Installation and Configuration system.
   - The Ericsson Network Resource Manager (NRM).
   - ``A flight booking system handling nearly a third of all airline tickets in the world'' (SICStus).
   - The java abstract machine specification is written in Prolog.
   - ...

