\title A Motivational Introduction

\begin{note} This is an interactive HTML version of the Course on logic and constraint logic programming: 1 - Motivation \end{note}

The Program Correctness Problem
THIS IS A TEST

@image{https://github.com/mherme/ald-tests/edit/main/Figs/logic_use_1.png}

THIS IS A TEST

Conventional models of using computers -- not easy to determine correctness!

Has become a very important issue, not just in safety-critical apps.

Components with assured quality, being able to give a warranty, ...

Being able to run untrusted code, certificate carrying code, ...

A Simple Imperative Program
Example:
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
Is it correct? With respect to what?
A suitable formalism is needed:
to provide specifications (describe problems), and
to reason about the correctness of programs (their implementation).
Natural Language
''Compute the squares of the natural numbers which are less or equal than 5.''

I
