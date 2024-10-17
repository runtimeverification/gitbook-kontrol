# A Comprehensive Guide to Writing Simplifications for Kontrol

## What are simplifications?

Symbolic execution in K in general and in Kontrol in particular is performed on a symbolic state consisting of: a **configuration**, which is a collection of cells; and a **path condition**, which is a set of Boolean constraints on the symbolic variables present in the configuration.

The purpose of simplifications, as their name suggests, is to simplify the job of the symbolic execution engine. They describe ways of getting the configuration and/or the path condition into a preferred format or state mathematical theorems that are either too complicated for the SMT solver to reason about both automatically and tractably or too simple for us to even want to call the SMT solver. Simplifications are K rewrite rules that have the `[simplification]` attribute, and their general form is:

```k
rule LHS => RHS requires C1 andBool ... andBool CN [simplification]
```

meaning that all occurrences of the term `LHS` in the current symbolic state should be replaced with the term `RHS` when the Boolean constraints `C1`, ..., `CN` are entailed by the current path condition `PC`, that is, when $\mathtt{PC} \implies \mathtt{C1} \wedge \ldots \wedge \mathtt{CN}$.

Importantly, a simplification is **sound** if and only if
$$
\mathtt{C1} \wedge \ldots \wedge \mathtt{CN} \implies \mathtt{(LHS \Leftrightarrow RHS)}
$$
meaning that the LHS and the RHS of the simplification have to be **equivalent** under assumptions `C1`, ..., `CN`, that is, that a simplification **must not lose information**. This is essential to keep in mind, especially since the K rewrite symbol `=>` can easily be misunderstood as implication in this context, and implications both can lose information and are commonly used in program verification (for instance, as forward consequence in various program logics). This means that, for example:
```k
rule A:Int >=Int 42 => A >=Int 0 [simplification]
```
would not be a sound simplification, whereas
```k
rule A:Int +Int 0 => A [simplification]
```
on the other hand, would.

### Simplifications by example

Let us now take a look at a number of illustrative examples of simplifications that Kontrol uses, starting from:
```k
rule [plus-group-conc]:
  (A +Int B) +Int C => A +Int (B +Int C)
  [simplification(40), symbolic(A), concrete(B, C)]
```
which is simple, yet fairly instructive. Let us examine its attributes, which tell us:
1. That the label/identifier of the simplification is `plus-group-conc`.
2. That the simplification is of priority 40, meaning that it is of higher priority than the default (which is 50). Assigning higher priority to simplifications allows us, for example, to put terms into a canonical form before applying other simplifications (see (S2) below), whereas assigning lower priority allows us to have last-resort simplification, only to be used if all others fail.
3. That the simplification is to be applied only if `A` is symbolic and `B` and `C` are concrete. This also reveals the purpose of the simplification, which is to simplify the addition by grouping the concrete operands together in order for them to be added up, producing a two-term addition instead of the three-term addition we started from.

At this point, we need to take a closer look at how the above simplification will be processed by the backend, revealing important principles that a Writer of Simplifications must remember.

1. The LHS of a simplification is matched **syntactically**. For example, for simplification `plus-group-conc` to be applied, we have to syntactically have a term of the form `(A +Int B) +Int C`, where `A` is symbolic and `B` and `C` are concrete.

2. Functions with all concrete parameters are **always evaluated before any other simplification is applied**. This means, for example, that the term `1 +Int 2` will always be evaluated to `3` without any other simplification applied to it beforehand. Interestingly, if we look at the simplification `plus-group-conc` above, this means that the attribute `symbolic(A)` is redundant. In particular, given that `B` already has to be concrete, if `A` were also concrete the expression `A +Int B` would get fully evaluated first and the term would no longer syntactically match the LHS of the simplification. In the development, we choose to keep the attribute `symbolic(A)` explicit for clarity. All of this tells us that when writing simplifications with a specific goal in mind, we have to ask ourselves: "What happens if some of the symbolic variables in my simplification were concrete?", and it turns out that sometimes further simplifications are required in such cases, as we will see later on.

3. The direction of simplification is **always bottom-up**. This means that the backend will simplify a given term starting from the atoms, moving left-to-right and bottom-to-top: for example, given the term `A +Int ((B +Int C) +Int D)`, the order of simplification would be: `A`, `B`, `C`, `B +Int C`, `D`, `((B +Int C) +Int D)`, `A +Int ((B +Int C) +Int D)`. The simplification passes are repeated for as long as one successful simplification or concrete evaluation can be performed. This means that we have to be careful not to introduce circular simplifications, an example of which would be:
```k
rule X +Int Y => Y +Int X [simplification]
```

Now, given all of the above, let us try to use `plus-group-conc` to simplify the term
```k
X +Int (((Y +Int 5) +Int 3) +Int Z)
```
First, going bottom-up, we spot that the (LHS of the) simplification can be matched against the sub-term `(Y +Int 5) +Int 3`, with the corresponding substitution equalling `{A: Y, B: 5, C: 3}`, and since the binding for `A` is symbolic and the bindings for `B` and `C` are concrete, we can apply the simplification to obtain
```k
X +Int ((Y +Int (5 +Int 3)) +Int Z)
```
In the next pass (always bottom-up), we first concretely evaluate `5 +Int 3`, obtaining
```k
X +Int ((Y +Int 8) +Int Z)
```
and then, as part of the same pass, we match the sub-term `(Y +Int 8) +Int Z` against `plus-group-conc` with the substitution `{A: Y, B: 8, C: Z}`, but cannot apply the simplification as the obtained binding for `C` is not concrete.

As our next example, we consider the simplification
```k
rule [mul-conc-left]:
  A *Int B => B *Int A
  [simplification(30), concrete(B)]
```
which **normalizes multiplication by moving concrete operands to the left**. First, note that this is not a circular simplification on its own. This is because, as discussed above, if `B` is concrete then for the simplification to be applied `A` must be symbolic. Once the simplification is applied, `A` will end up on the RHS of the multiplication, which must be concrete if the simplification is to be applied again, producing a contradiction. Second, this normalization choice allows us to minimize duplication of simplifications, as illustrated by the following simplification:
```k
rule [div-group-conc]:
  (A *Int B) /Int C => B *Int (A /Int C)
  requires notBool C ==Int 0 andBool A modInt C ==Int 0
  [simplification, concrete(A, C), preserves-definedness]
```
which, similarly to `plus-group-conc`, groups concrete operands for evaluation, but this time in the context of integer division, `/Int`. Unsurprisingly, the simplification can be applied only if `C` is different from zero, that is, if the division on the LHS is defined, and also if `C` divides `A`.

First, note that duplicating this simplification so that `B` is concrete instead of `A`:
```k
rule [div-group-conc-b]:
  (A *Int B) /Int C => A *Int (B /Int C)
  requires notBool C ==Int 0 andBool B modInt C ==Int 0
  [simplification, concrete(B, C), preserves-definedness]
```
is not necessary because of `mul-conc-left`. In fact, such a simplification would never be applied. To understand why this is so, consider, for example, the term
```k
(X *Int 6) /Int 2
```
to which we would expect `div-group-conc-b` to apply. However, given that simplifications are performed bottom-up, we will first get to apply `mul-conc-left` to `X *Int 6`, obtaining
```k
(6 *Int X) /Int 2
```
to which `div-group-conc` would then apply, but not `div-group-conc-b`.

Second, note that this simplification has the `preserves-definedness` attribute. This attribute should be used when either the LHS or the RHS contains a **partial function**, but we are certain that both the LHS and RHS are defined for all use cases. In this case, the partial function is `/Int` and both the LHS and the RHS are guaranteed to be defined given the `notBool C ==Int 0` requirement.

## Creating advanced simplifications: automating storage slot updates

In EVM, contract fields are stored in *[storage slots](https://docs.soliditylang.org/en/v0.8.24/internals/layout_in_storage.html)*, each of which holds 32 bytes, and  each of which may contain multiple fields, depending on their size. A general slot update is of the form
```k
(SHIFT:Int *Int VALUE:Int) |Int (MASK:Int &Int #asWord(SLOT:Bytes))
```
where:
- `SLOT` is a byte array representing the slot to be updated;
- `VALUE` is an integer representing the value to which a contract field in that slot should be updated;
- `MASK` is a bit-mask that is used to zero the part of the slot corresponding to the contract field to be updated; and
- `SHIFT` is a power of two that is used to shift the value so that it aligns with the part of the slot corresponding to the contract field to be updated.

For example, if we were updating a given 8-byte `uint64` field stored in a given slot starting from byte `8` (note that byte indices are computed from right to left), this would be achieved as follows, taking:
- `MASK ==Int 115792089237316195423570985008687907852929702298719625576012656144555070980095`, that is, `ffffffffffffffffffffffffffffffff0000000000000000ffffffffffffffff` in hex; and
- `SHIFT ==Int 2 ^Int 32`
```
32                      8        0
|xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx| : SLOT
|xxxxxxxxxxxxxxxx00000000xxxxxxxx| : MASK &Int #asWord(SLOT)

|000000000000000000000000yyyyyyyy| : VALUE
|0000000000000000yyyyyyyy00000000| : SHIFT *Int VALUE

|xxxxxxxxxxxxxxxxyyyyyyyyxxxxxxxx| : (SHIFT *Int VALUE) |Int (MASK &Int #asWord( SLOT:Bytes ))
```

Let us now understand how to automate this reasoning in Kontrol, recalling the general form:
```k
(SHIFT *Int VALUE) |Int (MASK &Int #asWord(SLOT))
```
and noting that `MASK` and `SHIFT` are always concrete, as they are generated by the Solidity compiler and are part of the contract bytecode. Therefore, our simplifications have to account for the cases in which at least one of `VALUE` and `SLOT` is symbolic.

### Identifying slot masks

We start by understanding how to capture slot masks correctly. A 32-byte (that is, 256-bit) integer `MASK` is a slot mask if and only if it is of the binary form:
```
|-------32 bytes / 256 bits-------|
|-REST-1s-||-WIDTH-0s-||-SHIFT-1s-|
```
where:
1. all bits of `SHIFT` and `REST`, if any, are ones;
2. all bits of `WIDTH` are zeros, and `WIDTH` is not of length zero; and
3. `SHIFT` and `WIDTH` are aligned to bytes, that is, their length is divisible by `8`.

Now, to identify the length of `SHIFT`, we could use an auxiliary function that returns the index of the first `0` bit of a given 256-bit integer, and to identify the length of `WIDTH`, we could similarly use a function that returns the index of the first `1` bit. We proceed to define these as follows:
```k
syntax Int ::= #getFirstOneBit(Int)  [function, total]
syntax Int ::= #getFirstZeroBit(Int) [function, total]

rule [gfo-succ]:
  #getFirstOneBit(X:Int) => log2Int(X &Int ((maxUInt256 xorInt X) +Int 1))
  requires #rangeUInt(256, X) andBool X =/=Int 0
  [preserves-definedness]

rule [gfo-fail]: #getFirstOneBit(_:Int) => -1 [owise]

rule [gfz-succ]:
  #getFirstZeroBit(X:Int) => #getFirstOneBit(maxUInt256 xorInt X)
  requires #rangeUInt(256, X)
  [preserves-definedness]

rule [gfz-fail]: #getFirstZeroBit(_:Int) => -1 [owise]
```
leaving the correctness check of the success cases as an exercise to the reader, and noting that we return `-1` to indicate failure when the argument is outside of the range `[0, pow256)` and also treat trying to find the first one-bit of `0` and the first zero-bit of `maxUInt256` as failures.

With these functions in place, we can now identify the length of `SHIFT` of a mask, in bits and in bytes, straightforwardly as follows:
```k
syntax Int ::= #getMaskShiftBits(Int)  [function, total]
syntax Int ::= #getMaskShiftBytes(Int) [function, total]

rule [gms-bits]: #getMaskShiftBits(X:Int) => #getFirstZeroBit(X)

rule [gms-bytes-succ]:
  #getMaskShiftBytes(X:Int) => #getFirstZeroBit(X) /Int 8
  requires #getMaskShiftBits(X) modInt 8 ==Int 0
  [preserves-definedness]

rule [gms-bytes-fail]: #getMaskShiftBytes(_:Int) => -1 [owise]
```
noting that if `#getMaskShiftBits` fails, so will `#getMaskShiftBytes` since `-1 modInt 8 =/=Int 0`.

Computing the length of `WIDTH`, on the other hand, is slightly more involved:
```k
syntax Int ::= #getMaskWidthBits(Int)  [function, total]
syntax Int ::= #getMaskWidthBytes(Int) [function, total]

rule [gmw-bits-succ-1]:
  #getMaskWidthBits(X:Int) => 256 -Int #getMaskShiftBits(X:Int)
  requires 0 <=Int #getMaskShiftBits(X)
   andBool 0 ==Int X >>Int #getMaskShiftBits(X)
   [preserves-definedness]

rule [gmw-bits-succ-2]:
  #getMaskWidthBits(X:Int) => #getFirstOneBit(X >>Int #getMaskShiftBits(X))
  requires 0 <=Int #getMaskShiftBits(X)
   andBool 0 <Int X >>Int #getMaskShiftBits(X)
   [preserves-definedness]

rule [gmw-bits-fail]: #getMaskWidthBits(_:Int) => -1 [owise]

rule [gmw-bytes-succ]:
  #getMaskWidthBytes(X:Int) => #getMaskWidthBits(X) /Int 8
  requires #getMaskWidthBits(X) modInt 8 ==Int 0
  [preserves-definedness]

rule [gmw-bytes-fail]: #getMaskWidthBytes(_:Int) => -1 [owise]
```
because we need to separately treat the case in which the `REST` part of the mask is empty, that is, when `#getFirstOneBit(X >>Int #getMaskShiftBits(X)) ==Int 0`. If we did not, then `#getFirstOneBit(X >>Int #getMaskShiftBits(X))` would return `-1` erroneously.

Now, we are able to state formally what it means for a 256-bit integer to be a valid slot mask:
```k
syntax Bool ::= #isMask(Int) [function, total]

rule [is-mask-true]:
  #isMask(X:Int) =>
    maxUInt256 ==Int
      X |Int ( 2 ^Int ( #getMaskShiftBits(X) +Int #getMaskWidthBits(X) ) -Int 1 )
  requires 0 <=Int #getMaskShiftBytes(X)
   andBool 0 <=Int #getMaskWidthBytes(X)
  [preserves-definedness]

rule [is-mask-false]: #isMask(_:Int) => false [owise]
```
Let us inspect the `is-mask-true` rule in more detail, in the light of the three requirements given above. Given that `0 <=Int #getMaskShiftBytes(X)` and `0 <=Int #getMaskWidthBytes(X)` hold by the requires clause, then:
1. all bits of `SHIFT`, if any, are ones;
2. all bits of `WIDTH` are zeros, and `WIDTH` is not of length zero; and
3. `SHIFT` and `WIDTH` are aligned to bytes
meaning that what is left is to show that all bits of `REST`, if any, are ones. This is achieved by the check
```k
maxUInt256 ==Int
  X |Int ( 2 ^Int ( #getMaskShiftBits(X) +Int #getMaskWidthBits(X) ) -Int 1 )
```
where, since having `( 2 ^Int ( #getMaskShiftBits(X) +Int #getMaskWidthBits(X) ) -Int 1 )` will set all of the bits corresponding to `SHIFT` and `WIDTH` to one, the only way for the bitwise-or to equal `maxUInt256` is if all the remaining bits, which correspond to `REST`, are also ones.

### Identifying value shifts

In addition to identifying masks, we also need to identify when `SHIFT` (as per the general slot update) is a valid shift, and this is when it is a byte-aligned power of two:
```k
syntax Bool ::= #isByteShift(Int) [function, total]

rule #isByteShift(X) => X ==Int 2 ^Int log2Int(X)
                andBool log2Int(X) modInt 8 ==Int 0
  requires 0 <Int X andBool X <Int pow256
  [preserves-definedness]

rule #isByteShift(_) => false [owise]
```

### Executing slot updates
Now we are in the position to write slot update simplifications. Before doing so, we recall their general form once more:
```
(SHIFT *Int VALUE) |Int (MASK &Int #asWord(SLOT))
```

#### Symbolic `SLOT`

First, we consider the case in which `SLOT` is symbolic, where we have to write a simplification that captures the effect of the mask, zeroing the appropriate part of the slot:
```k
// 1. Slot masking using &Int
// 32                               0
// |xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx| SLOT
// |--REST--||---WIDTH---||--SHIFT--| MASK
// ==================================
// |xxxxxxxxxx00000000000xxxxxxxxxxx|
rule [mask-b-and]:
  MASK:Int &Int SLOT:Int =>
    #asWord (
      #buf ( 32, SLOT )
      [ 32 -Int ( #getMaskShiftBytes(MASK) +Int #getMaskWidthBytes(MASK) )
          := #buf ( #getMaskWidthBytes(MASK), 0 ) ]
    )
    requires #isMask(MASK) andBool #rangeUInt(256, SLOT)
    [simplification, concrete(MASK), preserves-definedness]
```
and we unpack it in full detail. As slot updates are morally performed on byte arrays, we move from integers to byte arrays by constructing `#buf(32, SLOT)`, a 32-byte byte array that when interpreted as an unsigned integer yields `SLOT`. Note that `#buf(32, SLOT)` is well-defined due to the `#rangeUInt(256, SLOT)` hypothesis, which tells us that `SLOT` fits into 32 bytes. Then, we use the byte array update, `B [ I := B' ]`, which pastes byte array `B'` into byte array `B` starting from index `I`, to set `WIDTH` bytes to `0` starting from `32 -Int (SHIFT +Int WIDTH)`, which achieves the desired effect. Finally, we convert the byte array back to an integer using `#asWord`.

This, in general, results in a term of the form
```
#asWord ( B1 +Bytes ... +Bytes BI +Bytes            [P1]
          #buf(#getMaskWidthBytes(MASK), 0) +Bytes  [P2]
          BI+1 +Bytes ... +Bytes BN )               [P3]
```
where `+Bytes` denotes byte array concatenation, is right-associative, and is always normalized. Such a term, which we denote by `MASKED_SLOT`, can be divided into three parts:
1. [P1]: the part of the slot before the part to be updated (possibly empty): `B1 +Bytes ... +Bytes BI`;
2. [P2]: the part of the slot to be updated, now zeroed: `#buf(#getMaskWidthBytes(MASK), 0)`; and
3. [P3]: the rest of the slot (possibly empty): `BI+1 +Bytes ... +Bytes BN`.

This form, however, could end up being structured in less appealing ways if, for example, `BI` or `BI+1` are concrete. In that case, this built-in simplification
```k
rule [bytes-concat-left-assoc-conc]:
  B1:Bytes +Bytes (B2:Bytes +Bytes B3:Bytes) =>
    (B1 +Bytes B2) +Bytes B3
    [simplification(40), concrete(B1, B2), symbolic(B3)]
```
will bring them together into a single concrete byte array, obfuscating the zeroed part. In the worst case, when both [P1] and [P3] are concrete, the masking will result in just a concrete number.

Continuing, we are now dealing with a term of the form
```k
(SHIFT *Int VALUE) |Int MASKED_SLOT
```
and write simplifications that pinpoint into which part of a byte array concatenation the bitwise-or should propagate, should `MASKED_SLOT` have that form:
```k
// 2a. |Int and +Bytes, update propagates to left
rule [bor-update-to-left]:
  A |Int #asWord ( B1 +Bytes B2 ) =>
    #asWord ( #buf ( 32 -Int lengthBytes(B2), (A /Int (2 ^Int (8 *Int lengthBytes(B2)))) |Int #asWord ( B1 ) ) +Bytes B2 )
  requires #rangeUInt(256, A)
   andBool A modInt (2 ^Int (8 *Int lengthBytes(B2))) ==Int 0
   andBool lengthBytes(B1 +Bytes B2) <=Int 32
   [simplification(40), comm, preserves-definedness]

// 2b. |Int of +Bytes, update propagates to right
rule [bor-update-to-right]:
  A |Int #asWord ( B1 +Bytes B2 ) =>
    #asWord ( B1 +Bytes #buf ( lengthBytes(B2), A |Int #asWord ( B2 ) ) )
  requires 0 <=Int A
   andBool A <Int 2 ^Int (8 *Int lengthBytes(B2))
   andBool lengthBytes(B2) <=Int 32
   [simplification(40), comm, preserves-definedness]
```
In particular, the bitwise-or should propagate to the left if `A` (which here corresponds to `SHIFT *Int VALUE`) has no bits set in its last `LEN` bytes (where `LEN` denotes the length of `B2`), and to the right if `A` fits into `LEN` bytes.

Ideally, these simplifications would be sufficient for the general form of slot updates. However, as part of required `#asWord` and `#buf` reasoning, we also have the simplification
```k
rule [asWord-trim-leading-zeros]:
  #asWord ( BZ +Bytes B ) => #asWord ( B )
  requires #asInteger ( BZ ) ==Int 0
  [simplification(40), concrete(BZ), preserves-definedness]
```
that removes an arbitrary number of leading zeros from a byte array concatenation when under an `#asWord`. This means, in particular, that the zeros that are created by the masking could get absorbed into the next `+Bytes`-junct in the case when [P3] does not contain `+Bytes`. For example, if we had a slot with three pieces of information, `X`, `Y`, and `Z` as follows:
```k
[E1]: #asWord(#buf(10, X) +Bytes #buf(10, Y) +Bytes #buf(12, Z))
```
and we wanted to update `Y` to some value `W`, after the masking, and noting that `SHIFT ==Int 2 ^ 96`, we would have
```k
[E2]: ((2 ^Int 96) *Int W) |Int ( #asWord(#buf(10, X) +Bytes #buf(10, 0) +Bytes #buf(12, Z)) )
```
and after applying `bor-update-to-right`, we would have
```k
#asWord(#buf(10, X) +Bytes #buf(22, ((2 ^Int 96) *Int W) |Int #asWord (#buf(10, 0) +Bytes #buf(12, Z))))
```
but then `asWord-trim-leading-zeros` would fire, creating
```k
[E3]: #asWord(#buf(10, X) +Bytes #buf(22, ((2 ^Int 96) *Int W) |Int #asWord (#buf(12, Z))))
```
after which the following built-in simplification would fire
```k
rule [asWord-buf-inversion]:
  #asWord ( #buf ( WB:Int, X:Int ) ) => X
  requires 0 <=Int WB
   andBool 0 <=Int X andBool X <Int minInt ( 2 ^Int (8 *Int WB), pow256 )
   [simplification, concrete(WB), preserves-definedness]
```
resulting in
```k
[E4]: #asWord(#buf(10, X) +Bytes #buf(22, ((2 ^Int 96) *Int W) |Int Z))
```

To account for these cases, we now need the following simplification:
```k
// 32                               0
// |0000000000000000000000yyyyyyyyyy| Y
// |00000000000000000000010000000000| SHIFT
// ==================================
// |xxxxxxxxxxxxxxxxxxxxxxyyyyyyyyyy|
rule [bor-update-with-shift]:
  ( SHIFT *Int X ) |Int Y =>
    #asWord ( #buf( 32 -Int ( log2Int(SHIFT) /Int 8 ), X ) +Bytes
              #buf( log2Int(SHIFT) /Int 8, Y ) )
  requires #isByteShift(SHIFT)
    andBool 0 <=Int X andBool X <Int 2 ^Int (8 *Int (32 -Int ( log2Int(SHIFT) /Int 8 )))
    andBool 0 <=Int Y andBool Y <Int SHIFT
    [simplification(42), concrete(SHIFT), comm, preserves-definedness]
```
which structures the (part of the) slot by putting `X` into the bytes above the `SHIFT` if `Y` is unaffected by the bitwise-or. When this rule fires on our example, we get:
```k
[E5]: #asWord(#buf(10, X) +Bytes #buf(22, #asWord(#buf(20, W) +Bytes #buf(12, Z))))
```
which will require some further massaging later. Right now, it is important to note that we also have to have another rule for this case, in which the `SHIFT` is not explicit because the updating value is concrete:
```k
rule [bor-update-without-shift]:
  X |Int Y =>
    #asWord ( #buf ( 32 -Int #getFirstOneBit(X) /Int 8, X /Int ( 2 ^Int ( 8 *Int ( #getFirstOneBit(X) /Int 8 ) ) ) ) +Bytes
              #buf ( #getFirstOneBit(X) /Int 8, Y ) )
  requires #rangeUInt(256, X) andBool 0 <=Int #getFirstOneBit(X)
   andBool 0 <=Int Y andBool Y <Int 2 ^Int ( 8 *Int ( #getFirstOneBit(X) /Int 8 ) )
   [simplification(42), concrete(X), preserves-definedness]
```
and in which we require that the first one-bit of `X` (which we can compute, as `X` is concrete), is in a byte that is higher than the bytes occupied by `Y`.

Returning to [E5], we note that we have this term:
```k
#buf(22, #asWord (#buf(20, W) +Bytes #buf(12, Z)))
```
where a buffer of size 22 contains the value described by a buffer of size 32. This is a result of the padding-to-32-bytes introduced by `bor-update-with-shift` and `bor-update-without-shift`, which is needed since we do not know a priori how wide `VALUE` is. To reconcile these buffer sizes, we have the following simplification:
```k
rule [buf-asWord-crop]:
  #buf (W:Int , #asWord(B:Bytes)) => #range(B, lengthBytes(B) -Int W, W)
  requires 0 <=Int W andBool W <=Int 32 andBool W <Int lengthBytes(B)
   andBool #asWord ( #range(B, 0, lengthBytes(B) -Int W) ) ==Int 0
  [simplification, concrete(W), preserves-definedness]
```
which isolates the part of the longer buffer that fits into the shorter one, assuming the value in it indeed fits the shorter one. When this simplification fires on [E5], we obtain
```k
[E6]: #asWord(#buf(10, X) +Bytes #range(#buf(20, W) +Bytes #buf(12, Z), 10, 22))
```
which then, through the following built-in simplification
```k
rule [range-inside-cHead-concat]:
  #range(B1:Bytes +Bytes B2:Bytes, S:Int, W:Int) =>
    #range(#range(B1, S, lengthBytes(B1) -Int S) +Bytes B2, 0, W)
  requires 0 <Int S andBool S <=Int lengthBytes(B1)
  [simplification]
```
becomes
```k
[E7]: #asWord(#buf(10, X) +Bytes #range(#range(#buf( 20, W ), 10, 10) +Bytes #buf(12, Z), 0, 22))
```
then, through the following built-in simplification
```k
rule [range-buf-value]:
  #range (#buf(W1:Int, X:Int), S2:Int, W2:Int) => #buf(W2, X)
  requires 0 <=Int X andBool X <Int 2 ^Int (8 *Int W2)
   andBool 0 <=Int S2 andBool 0 <=Int W2 andBool W1 ==Int S2 +Int W2
  [simplification, concrete(W1, S2, W2), preserves-definedness]
```
simplifes to
```k
[E8]: #asWord(#buf(10, X) +Bytes #range(#buf(10, W) +Bytes #buf(12, Z), 0, 22))
```
and which then, through a number of further built-in `#range`-related simplifications, simplifies to the desired
```
[E9]: #asWord(#buf(10, X) +Bytes #buf(10, W) +Bytes #buf(12, Z))
```

We conclude with the final piece of the slot update puzzle, which is an additional simplification that splits the shift from the value in the case that the part of the slot being processed during the update is zero-valued (for example, if `Z ==Int 0` in the running example above):
```k
rule [buf-split-on-shift]:
  #buf ( W, SHIFT *Int X ) =>
    #buf( W -Int ( log2Int(SHIFT) /Int 8 ), X ) +Bytes #buf( log2Int(SHIFT) /Int 8, 0)
  requires 0 <=Int W andBool W <=Int 32 andBool #isByteShift(SHIFT)
   andBool 0 <=Int X andBool X <Int 2 ^Int (8 *Int (W -Int ( log2Int(SHIFT) /Int 8)))
   [simplification, concrete(W, SHIFT), preserves-definedness]
```