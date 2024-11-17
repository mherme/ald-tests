\title Teaching Pure LP with Prolog and a Fair Search Rule

@comment{Small kludge to start already in slides mode (needs to be improved)}
```ciao_runnable
%! \begin{jseval}
async(pgcell) => { pgcell.pgset.main_doc.set_slide_mode(true); }
%! \end{jseval}
```

# The vision is easy to convey at first 

- Some fun examples - circuit topology.

@p @image{https://github.com/mherme/ald-tests/blob/main/teachingws/Figs/logic_circuit_color_small}


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

