# A detailed look at rules and rule application in K

In general, any semantics written in K works on a symbolic state of the form `(C, PC)`, where:
1. `C` is the **configuration**, which we can think of as a collection of cells; and
2. `PC` is the a **path condition**, which is a set of Boolean constraints on the symbolic variables present in the configuration

and allows the creator to use three flavours of rules:
1. **rewrite rules**, which describe the transitions of the semantics;
2. **function evaluation rules** (onward: function rules), which describe how functions are evaluated; and
3. **simplifications**, which allow one to use mathematical theorems to simplify terms.

For the purposes of this tutorial, we can assume that all three types of rules have the form:
```k
rule LHS => RHS
  requires R1 andBool ... andBool RN
   ensures E1 andBool ... andBool EM
```
where:
- `RI` and `EI`, for `1 <=Int I <=Int N`, are propositional formulae;
- the `LHS` of a rewrite rule does not contain function symbols;
- the `LHS` and `RHS` of function rules and simplifications are propositional formulae;
- the top-level symbol of the `LHS` of a function rule or a simplification is a function symbol.

In principle, the rewrite rules can contain multiple rewrites (`=>`), but as all can be recast to fit the above form there is no loss of generality.

Let us now understand what it means to apply a given rewrite rule to a given symbolic state `(C, PC)`. This process consists of the following stages:
1. **Matching**: first, the backend understands whether or not the configuration `C` can be *syntactically matched* against the `LHS` of the rule. This phase is purely syntactic and does not involve the SMT solver. If successful, it yields a substitution `S`, which maps symbolic variables in the `LHS` to the corresponding parts of `C`. Otherwise, the rule cannot be applied. Contrast, for example, the following two rules, where the first rule requires a `0` syntactically in second position, whereas the second rule requires there a term that is semantically equivalent to `0`:
```k
R1: rule <k> A : 0 : B : WS => A +Int B : WS ... <k>
R2: rule <k> A : X : B : WS => A +Int B : WS ... <k> requires X ==Int 0
```

2. **Requires**: next, the backend applies the substitution `S` to the requires clause, obtaining the first-order constraints `S(RI)`, for `1 <=Int I <=Int N`. Then, it checks, in unspecified order, whether `PC /\ S(RI)` is *satisfiable*, for each `RI`, fully applying simplifications and using the SMT solver. If this holds, the rule can be applied. Otherwise, it cannot.
3. **Application**: next, the rule is actually applied, meaning that the part of the term `C` that was identified to match the `LHS` of the rule is replaced by `S(RHS)`.
4. **Ensures**: finally, the substitution is applied to the `ensures` clause, obtaining `S(EI)`, for `1 <=Int I <=Int M`, which are then fully simplified and the non-trivial ones added to the path condition. Finally, an SMT-check is performed to understand whether or not this has made the path condition unsatisfiable, and if this is the case, the obtained symbolic state is marked as *vacuous*.

One full rewrite step of a proof generalises the above process in roughly the following way:
1. It takes the current symbolic state `(C, PC)` and attempts to apply all of the rewrite rules available from the language definition, in order of priority.
2. If no rules are applicable:
   1. If no lower priority rules are available, the state is marked as *stuck*.
   2. Otherwise, step 1 is repeated with the next set of lower priority rules.
3. Otherwise, let there be `M` applicable rules, let their requires clauses be `RI`, for `1 <=Int I <=Int M`, and let the obtained substitutions be `SI`, for `1 <=Int I <=Int M`. The backend then checks whether or not the applicable rules cover the space of possibilities, that is, whether or not `PC /\ !(S1(R1) \/ ... \/ SM(RM))`, that is, `PC /\ !S1(R1) /\ ... /\ !SM(RM)` is satisfiable. This path condition characterises what is known as *the remainder branch*. If this branch is feasible, the process is repeated for it, but only for the lower priority rules.

Like rewrite rules, simplifications and function evaluation rules are applied in order of priority. However, they are applied on a first-success basis and their `requires` clauses are checked using *entailment*, not satisfiability. These rules are meant to fire only if we know with certainty that they apply, that is, they are not meant to create branches in the proof.

### An example of rule application

Let us consider the following symbolic state
```k
<k> #execute ~> #halt </k>
<wordStack> X1:Int +Int X2:Int : Y1:Int +Int Y2:Int : Z : .WordStack </wordStack>
<output> 0 </output>
#And { true #Equals 0 <=Int X1 }
#And { true #Equals 0 <=Int X2 }
```
and try to apply the following rules:
```k
rule [exec-b-01]:
  <k> #execute => . ... </k>
  <wordStack> X : _ : Y : WS => WS </cx>
  <output> _ => X *Int Y </output>
  requires 0 <=Int X andBool X <Int 1000
  [priority(40)]

rule [exec-b-02]:
  <k> #execute => . ... </k>
  <wordStack> X : _ : Y : WS => WS </cx>
  <output> _ => X /Int Y </output>
  requires 1000 <=Int X andBool X <Int 2000 andBool notBool Y ==Int 0
  [priority(40)]

rule [exec-b-03]:
  <k> #execute => . ... </k>
  <wordStack> X : Y : Z : WS => WS </cx>
  <output> _ => X +Int Y +Int Z </output>
  requires 0 <=Int X
```

We start by trying to apply the higher priority rules, `exec-b-01` and `exec-b-02`, at priority 40. Both rules apply, with the same substitution
```k
S1/S2: { X |-> X1 +Int X2; Y |-> Z; WS |-> .WordStack }
```
and the corresponding substituted requires clauses:
```k
S1(R1): 0 <=Int X1 +Int X2 andBool X1 +Int X2 <Int 1000
S2(R2): 1000 <=Int X1 +Int X2 andBool X1 +Int X2 <Int 2000 andBool notBool Z ==Int 0
```
Next, we check if the space of possibilities is covered by computing the remainder condition `PC /\ !S1(R1) /\ S2(R2)`:
```k
0 <=Int X1 andBool 0 <=Int X2 andBool
( X1 +Int X2 <Int 0 orBool X1 +Int X2 >=Int 1000 ) andBool
( X1 +Int X2 <Int 1000 orBool X1 +Int X2 >=Int 2000 orBool Z ==Int 0 )
```
which is equivalent to
```k
0 <=Int X1 andBool 0 <=Int X2 andBool X1 +Int X2 >=Int 1000 andBool
( X1 +Int X2 >=Int 2000 orBool Z ==Int 0 )
```
and which is satisfiable, which means that we have to try the `exec-b-03` rule as well, which also can be applied, with the substitution:
```k
S3: { X |-> X1 +Int X2; Y |-> Y1 +Int Y2; Z |-> Z; WS |-> .WordStack }
```
and the substituted requires clause
```k
S3(R3): 0 <=Int X1 +Int X2
```
This time, the remainder condition equals
```k
0 <=Int X1 andBool 0 <=Int X2 andBool X1 +Int X2 >=Int 1000 andBool
( X1 +Int X2 >=Int 2000 orBool Z ==Int 0 ) andBool notBool ( 0 <=Int X1 +Int X2 )
```
but that is not satisfiable, meaning that the three rules cover the space of possibilities given the initial symbolic state. This means that we end up with three branches, resulting in the three following symbolic states:
```k
<k> #halt </k>
<wordStack> .WordStack </wordStack>
<output> ( X1 +Int X2 ) *Int Z </output>
#And { true #Equals 0 <=Int X1 }
#And { true #Equals 0 <=Int X2 }
#And { true #Equals X1 +Int X2 <Int 1000 }

<k> #halt </k>
<wordStack> .WordStack </wordStack>
<output> ( X1 +Int X2 ) /Int Z </output>
#And { true #Equals 0 <=Int X1 }
#And { true #Equals 0 <=Int X2 }
#And { true #Equals 1000 <=Int X1 +Int X2 }
#And { true #Equals X1 +Int X2 <Int 2000 }
#And #Not { Z #Equals 0 }

<k> #halt </k>
<wordStack> .WordStack </wordStack>
<output> X1 +Int X2 +Int Y1 +Int Y2 +Int Z </output>
#And { true #Equals 0 <=Int X1 }
#And { true #Equals 0 <=Int X2 }
#And { true #Equals X1 +Int X2 >=Int 1000 }
#And ( { true #Equals # X1 +Int X2 >=Int 2000 } #Or { Z #Equals 0 } )
```

## Aside: writing a well-formed K semantics

There are two principles that should be followed in order for a K semantics to be well-formed, that is, that starting from a well-defined initial symbolic state and executing the associated rewrite rules, function evaluation rules, and simplifications can never end up in an undefined state:

1. All rewrite rules must cover the entire space of possibilites. Some examples of this coverage from KEVM are the rules for the `JUMPI` instruction:
```k
rule [jumpi.false]: <k> JUMPI _DEST I => .K        ... </k> requires         I ==Int 0
rule [jumpi.true]:  <k> JUMPI  DEST I => JUMP DEST ... </k> requires notBool I ==Int 0
```
where the space of possibilities is split into two at the level of the path condition, and the rules for the `BALANCE` instruction:
```k
rule <k> BALANCE ACCT => #accessAccounts ACCT ~> BAL ~> #push ... </k>
     <account>
       <acctID> ACCT </acctID>
       <balance> BAL </balance>
       ...
     </account>

rule <k> BALANCE ACCT => #accessAccounts ACCT ~> 0 ~> #push ... </k> [owise]
```
where the space of possibilities is split into two at the level of the configuration, with the help of the `owise` attribute.

2. Any rewrite rules, function rules, or simplifications that use partial functions on their RHS (for example, `/Int` and `modInt`) must ensure that the parameters of said functions are within their domain. In addition, all of these rules must be marked with the `preserves-definedness` attribute.