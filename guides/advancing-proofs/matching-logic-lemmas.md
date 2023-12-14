---
description: The deepest layer of Kontrol reasoning
---

# Matching Logic Lemmas

Similarly, as in the last section, to ensure reproducibility, we recommend fixing the **Kontrol** version to 0.1.12 with `kup install kontrol --version v0.1.12`.

[Matching Logic](http://www.matching-logic.org/) is the underlying mathematical foundation of the [**K** framework](https://kframework.org/) and, by extension, [**KEVM**](https://jellopaper.org/). To operate **Kontrol**, it is not necessary to understand ML in depth. However, it is useful to know _what_ ML is when reading **Kontrol** outputs. Sometimes, reasoning steps must be provided at the ML level, as we will see in this case.

The ML operators are defined [here](https://github.com/runtimeverification/k/blob/master/k-distribution/include/kframework/builtin/kast.md?plain=1#L109-L165), but the ones we'll be encountering and reasoning about are the following:

{% code fullWidth="true" %}
```
syntax {Sort} Sort ::= "#Top" [klabel(#Top), symbol, group(mlUnary)]
                       | "#Bottom" [klabel(#Bottom), symbol, group(mlUnary)]
                       | "#True" [klabel(#Top), symbol, group(mlUnary), unparseAvoid]
                       | "#False" [klabel(#Bottom), symbol, group(mlUnary), unparseAvoid]
                       | "#Not" "(" Sort ")" [klabel(#Not), symbol, mlOp, group(mlUnary)]
                       
syntax {Sort} Sort ::= Sort "#And" Sort [klabel(#And), symbol, assoc, left, comm, unit(#Top), mlOp, group(mlAnd), format(%i%1%d%n%2%n%i%3%d)]
                       > Sort "#Or" Sort [klabel(#Or), symbol, assoc, left, comm, unit(#Bottom), mlOp, format(%i%1%d%n%2%n%i%3%d)]
                       > Sort "#Implies" Sort [klabel(#Implies), symbol, mlOp, group(mlImplies), format(%i%1%d%n%2%n%i%3%d)]
                       
syntax {Sort1, Sort2} Sort2 ::= "{" Sort1 "#Equals" Sort1 "}" [klabel(#Equals), symbol, mlOp, group(mlEquals), comm, format(%1%i%n%2%d%n%3%i%n%4%d%n%5)]
```
{% endcode %}

Notice how both `#True` and `#False` are internally represented as `#Top` and `#Bottom` respectively, since they respectfully have `#Top` and `#Bottom` as their `klabel`, plus the `symbol` [attribute](https://kframework.org/docs/user\_manual/#klabel\(\_\)-and-symbol-attributes).

These symbols are not boolean symbols, but they may appear in conjunction with booleans. For instance, we can have statements of the sort `#Not ( { X #Equals true } )` where `X` is a boolean value, and `true` is the boolean true value.

## Verifying Solady's `mulWadUp`

In this example, we'll verify [Solady's (`mulWadUp`) function](https://github.com/Vectorized/solady/blob/16c0dda5838fbb1350a0735dffa564b0b0a11e7e/src/utils/FixedPointMathLib.sol#L67), defined as follows

```solidity
function mulWadUp(uint256 x, uint256 y) internal pure returns (uint256 z) {
    assembly {
        if mul(y, gt(x, div(not(0), y))) {
            mstore(0x00, 0xbac65e5b) // `MulWadFailed()`.
            revert(0x1c, 0x04)
        }
        z := add(iszero(iszero(mod(mul(x, y), WAD))), div(mul(x, y), WAD))
    }
}
```

The description of `mulWadUp` is "Equivalent to `(x * y) / WAD` rounded up." That is, rounding up to the least significant digit. To this end, we define the following property test

```solidity
function testMulWadUp(uint256 x, uint256 y) public {
    if(y == 0 || x <= (type(uint256).max)/y) {
        uint256 zSpec = ((x * y)/WAD)*WAD < x * y ? (x * y)/WAD + 1 : (x * y)/WAD;
        uint256 zImpl = FixedPointMathLib.mulWadUp(x, y);
        assertEq(zImpl, zSpec);
    } else {
        vm.expectRevert(); // FixedPointMathLib.MulWadFailed.selector
        FixedPointMathLib.mulWadUp(x, y);
    }
}
```

This property test states that if `y` is zero or  the product `x*y` doesn't overflow, the result of `mulWadUp(x, y)` should be equal to `zSpec`. Otherwise, it should revert.\
Let's analyze `zSpec`. If `x*y` is not multiple of `WAD` (1e18), then `zSpec` will be `(x * y)/WAD + 1`, rounding up `(x * y) / WAD`. Otherwise `zSpec` will be `(x * y)/WAD` since the division is exact.

After symbolically executing this test, the following message appears, indicating to us that Kontrol could not prove that the property holds for every possible input

{% code overflow="wrap" fullWidth="true" %}
```
1 Failure nodes. (0 pending and 1 failing)

Failing nodes:

  Node id: 21
  Failure reason:
    Structural matching failed, the following cells failed individually (antecedent #Implies consequent):
    K_CELL: ( JUMPI 4500 bool2Word ( ( notBool ( ( notBool ( ( VV0_x_114b9705:Int *Int VV1_y_114b9705:Int ) /Int 1000000000000000000 ) ==Int 0 ) andBool maxUInt256 /Word ( ( VV0_x_114b9705:Int *Int VV1_y_114b9705:Int ) /Int 1000000000000000000 ) <Int 1000000000000000000 ) ) )
    ~> #pc [ JUMPI ]
    ~> #execute #Implies #halt )
    ~> CONTINUATION:K
    ACCOUNTS_CELL: ( <account>
      <balance>
        ( 0 #Implies ACCT_BALANCE_FINAL:Int )
      </balance>
      <storage>
        ( .Map #Implies ACCT_STORAGE_FINAL:Map )
      </storage>
      <origStorage>
        ( .Map #Implies ACCT_ORIGSTORAGE_FINAL:Map )
      </origStorage>
      <nonce>
        ( 1 #Implies ACCT_NONCE_FINAL:Int )
      </nonce>
      ...
    </account>
    ( ( ACCOUNTS_FINAL:AccountCellMap
    ... ) #Implies ... ) )
  Path condition:
    ( { true #Equals ( notBool VV1_y_114b9705:Int ==Int 0 ) }
#And { true #Equals ( notBool ( maxUInt256 /Int VV1_y_114b9705:Int ) <Int VV0_x_114b9705:Int ) } )
```
{% endcode %}

If we look at the **K** configuration at node 21, we can inspect the term and path constraints to understand better why the symbolic execution didn't succeed

<figure><img src="../../.gitbook/assets/Screenshot from 2023-11-03 13-30-46.png" alt=""><figcaption><p>Problematic section of first running attempt</p></figcaption></figure>

Notice that node 21 is `stuck`, unlike in the previous example, where it fully traversed an unfeasible branch.

If we look at the `<k>` cell, we can see that what's being evaluated is an overflow check introduced by the Solidity compiler. After parsing the `bool2Word` condition, we get the following expression

{% code overflow="wrap" fullWidth="false" %}
```
bool2Word ( ( 
        notBool ( 
                ( notBool ( ( X:Int *Int Y:Int ) /Int WAD ) ==Int 0 ) 
                andBool 
                maxUInt256 /Word ( ( X:Int *Int Y:Int ) /Int WAD ) <Int WAD 
                )
 ) )
```
{% endcode %}

This overflow check comes from the conditional in the `zSpec` assignment:

```solidity
uint256 zSpec = ((x * y)/WAD)*WAD < x * y ? (x * y)/WAD + 1 : (x * y)/WAD;
```

In this case, the overflow check is introduced for the rightmost multiplication of `((x * y)/WAD)*WAD`.&#x20;

Next, we should inspect the path constraints to understand why **Kontrol** couldn't make any reasoning progress. The conditions that should catch our attention are the following:
