---
description: How to debug your KCFG and find KEVM reasoning gaps
---

# KEVM Lemmas

In this section, we will verify [Solady's](https://github.com/Vectorized/solady) `mulWad` and `mulWadUp` functions, demonstrating how to identify and write good lemmas. To be consistent while executing these examples, we will update and fix **Kontrol** to `version 0.1.12`. To install it quickly run the following:

```
kup install kontrol --version v0.1.12
```

The reasoning engine behind **Kontrol** is the **K** framework and the **K** definition of the EVM semantics, **KEVM**. Sometimes it is necessary to address reasoning gaps or suggest simplification strategies at the **KEVM** level. To gain a better understanding of the definitions and rules involved in expressions that are not being simplified as desired, you can explore the [**KEVM** repository](https://github.com/runtimeverification/evm-semantics).

## Verifying Solady's `mulWad`

{% hint style="info" %}
Note: `WAD = 10**18`
{% endhint %}

For this example, we'll verify Solady's [`mulWad`](https://github.com/Vectorized/solady/blob/16c0dda5838fbb1350a0735dffa564b0b0a11e7e/src/utils/FixedPointMathLib.sol#L54)function, defined as:

```solidity
function mulWad(uint256 x, uint256 y) internal pure returns (uint256 z) {
    assembly {
        if mul(y, gt(x, div(not(0), y))) {
            mstore(0x00, 0xbac65e5b)
            revert(0x1c, 0x04)
        }
        z := div(mul(x, y), WAD)
    }
}
```

Our goal is to demonstrate that this is equivalent to `(x * y) / WAD` for any two integers `x` and `y`. To achieve this, we define the following property test:

```solidity
function testMulWad(uint256 x, uint256 y) public {
    if(y == 0 || x <= type(uint256).max / y) {
        uint256 zSpec = (x * y) / WAD;
        uint256 zImpl = FixedPointMathLib.mulWad(x, y);
        assertEq(zImpl, zSpec);
    } else {
        vm.expectRevert();
        FixedPointMathLib.mulWad(x, y);
    }
}
```

This property test asserts that if the product `x*y` does not result in an overflow, _or_ if `y` is zero, then output of `mulWad(x, y)` should be equal to `(x*y)/WAD`. However, if an overflow occurs, the function `mulWad` should revert.

After symbolically executing this test, the generated message indicates that **Kontrol** was unable to prove that the property holds for all possible inputs. This output will look like the following:

{% code fullWidth="true" %}
```
1 Failure nodes. (0 pending and 1 failing)

Failing nodes:

  Node id: 23
  Failure reason:
    Implication check failed, the following is the remaining implication:
    ( ( #Not ( { VV1_y_114b9705:Int #Equals 0 } )
    #And ( { true #Equals 0 <=Int CALLER_ID:Int }
    #And ( { true #Equals 0 <=Int ORIGIN_ID:Int }
    #And ( { true #Equals 0 <=Int NUMBER_CELL:Int }
    #And ( { true #Equals 0 <=Int VV0_x_114b9705:Int }
    #And ( { true #Equals 0 <=Int VV1_y_114b9705:Int }
    #And ( { true #Equals CALLER_ID:Int <Int pow160 }
    #And ( { true #Equals ORIGIN_ID:Int <Int pow160 }
    #And ( { true #Equals NUMBER_CELL:Int <=Int maxSInt256 }
    #And ( { true #Equals VV0_x_114b9705:Int <Int pow256 }
    #And ( { true #Equals VV1_y_114b9705:Int <Int pow256 }
    #And ( { true #Equals VV0_x_114b9705:Int <=Int ( maxUInt256 /Int VV1_y_114b9705:Int ) }
    #And ( { false #Equals ( ( notBool VV0_x_114b9705:Int ==Int 0 ) andBool maxUInt256 /Word VV0_x_114b9705:Int <Int VV1_y_114b9705:Int ) }
    #And #Not ( { 0 #Equals chop ( ( VV1_y_114b9705:Int *Int bool2Word ( ( maxUInt256 /Int VV1_y_114b9705:Int ) <Int VV0_x_114b9705:Int ) ) ) } ) ) ) ) ) ) ) ) ) ) ) ) ) ) #Implies { true #Equals foundry_success ( ... statusCode: EVMC_REVERT , failed: #lookup ( .Map , 46308022326495007027972728677917914892729792999299745830475596687180801507328 ) , revertExpected: false , opcodeExpected: false , recordEventExpected: false , eventExpected: false ) } )
  Path condition:
    ( { true #Equals ( notBool VV1_y_114b9705:Int ==Int 0 ) }
#And ( { true #Equals ( notBool ( maxUInt256 /Int VV1_y_114b9705:Int ) <Int VV0_x_114b9705:Int ) }
#And { true #Equals ( notBool chop ( ( VV1_y_114b9705:Int *Int bool2Word ( ( maxUInt256 /Int VV1_y_114b9705:Int ) <Int VV0_x_114b9705:Int ) ) ) ==Int 0 ) } ) )
```
{% endcode %}

Our next step is to inspect the **KCFG** (`kontrol view-kcfg`) to understand which path condition is leading to node 23. To learn more about the **KCFG:** [k-control-flow-graph-kcfg.md](../kevm-foundry-integration-example/k-control-flow-graph-kcfg.md "mention")

## Identifying branching conditions

When we inspect the branching condition that leads to the failing node, we can see that it corresponds to the `if` statement of the `mulWad` function.

<figure><img src="../../.gitbook/assets/Screenshot from 2023-10-19 16-08-42.png" alt=""><figcaption><p>Branching condition leading to failing node</p></figcaption></figure>

Let's unparse the branching condition. The condition is:

{% code overflow="wrap" fullWidth="true" %}
```
chop ( ( VV1_y_114b9705:Int *Int bool2Word ( ( maxUInt256 /Int VV1_y_114b9705:Int) <Int VV0_x_114b9705:Int ) ) ) ==Int 0
```
{% endcode %}

Taking into account that `chop(x)` is  `(x)mod[2**256]`, we can write the expression in a more palatable way:

{% code overflow="wrap" %}
```
(y *Int bool2Word((maxUInt256 /Int y) <Int x)) modInt 2**256 ==Int 0
```
{% endcode %}

There are three main things to know about this condition.&#x20;

* The operands `*Int`, `<Int`, `modInt`, `/Int`, and `==Int` are **K** functions doing the obvious thing, with the suffix `Int` indicating the type of their arguments
* The function `bool2Word` takes a boolean value and converts it into an EVM word. Particularly, it converts `true` to `1` and `false` to `0`. You can find the definition [here](https://github.com/runtimeverification/evm-semantics/blob/master/kevm-pyk/src/kevm\_pyk/kproj/evm-semantics/evm-types.md#boolean-conversions).
* The modulo operation appears because, inside assembly blocks, the Solidity compiler doesn't insert overflow checks

Thus, the condition is equivalent to `y == 0 || types(uint256).max / y  >= x`. Now, since this is the condition to branch on the first `if` of the `mulWad` function, **Kontrol** will explore the two different branches, one asserting that `(y *Int bool2Word((maxUInt256 /Int y) <Int x)) modInt 2**256 ==Int 0` holds, and another one asserting that the opposite holds (via the `notBool` operator).

The branch starting on node 21 explores the path of negating `(y *Int bool2Word((maxUInt256 /Int y) <Int x)) modInt 2**256 ==Int 0`. That is, entering the `if` statement, which is why we see that node 23 has `statusCode: EVMC_REVERT`.

## Dealing with path constraints

Having a node with a status code `EVMC_REVERT` can mean that either there's a bug and that revert should not be happening, or that the branch is unfeasible but **Kontrol** did not realize it. Let's look at the path conditions of node 21 to see if a contradiction can be derived.

<figure><img src="../../.gitbook/assets/Screenshot from 2023-10-19 17-01-56.png" alt=""><figcaption><p>Path conditions (constraints) of node 21</p></figcaption></figure>



If we look a the first condition and the ones at the end, we can spot something suspicious. Let's inspect the following conditions:

1. `#Not ( { VV1_y_114b9705:Int #Equals 0 } )`
2. `{ VV0_x_114b9705:Int <=Int ( maxUInt256 /Int VV1_y_114b9705:Int ) #Equals true }`
3. `( notBool chop ( ( VV1_y_114b9705:Int *Int bool2Word ( ( maxUInt256 /Int VV1_y_114b9705:Int ) <Int VV0_x_114b9705:Int ) ) ) ==Int 0 )`

After renaming the Matching Logic operators and other syntactic noise, the three conditions are

1. `y != 0`
2. `x <= maxUInt256 / y`
3. `y * bool2Word(maxUInt256 / y < x)) != 0`

Given these constraints, we can see how 1 and 2 imply that condition 3 is false. Thus we're not facing a bug in `mulWad`, but rather a reasoning gap in Kontrol.

## Bridging the (reasoning) gap

**Kontrol** is able to infer `maxUInt256 / y < x = false` from conditions 1 and 2, and a quick `git grep -rin 'rule bool2Word'` in the [evm-semantics](https://github.com/runtimeverification/evm-semantics/tree/master/kevm-pyk/src/kevm\_pyk/kproj/evm-semantics) definition tells us that these are the defined rewrite rules for `bool2Word`:

{% code fullWidth="true" %}
```
evm-types.md:32:     rule bool2Word( true  ) => 1
evm-types.md:33:     rule bool2Word( false ) => 0

lemmas/lemmas.k:50:  rule bool2Word(A) |Int bool2Word(B) => bool2Word(A  orBool B) [simplification]
lemmas/lemmas.k:51:  rule bool2Word(A) &Int bool2Word(B) => bool2Word(A andBool B) [simplification]

lemmas/lemmas.k:56:  rule bool2Word(_B) |Int 1 => 1            [simplification]
lemmas/lemmas.k:57:  rule bool2Word( B) &Int 1 => bool2Word(B) [simplification]

lemmas/lemmas.k:174: rule bool2Word(X ==Int 1)        => X requires #rangeBool(X) [simplification]
lemmas/lemmas.k:178: rule bool2Word( B:Bool ) ==Int I => B ==K word2Bool(I)       [simplification, concrete(I)]
```
{% endcode %}

Knowing the existing rules for any function is important since this means knowing how the function is treated by **KEVM** and thus by **Kontrol**.

Note that none of those rules is telling **Kontrol** (actually, **KEVM**) to attempt to simplify the boolean arguments of `bool2Word`. Because none of these rules really apply to condition 3, no simplification is made. This is why **Kontrol** derives no contradiction from condition 3 above, it's treating that expression as purely symbolic. To tell **Kontrol** to attempt to simplify the boolean arguments of the `bool2Word` function, we need to add the following rules to the file described in the last section

```
rule bool2Word ( X ) => 1 requires X         [simplification]
rule bool2Word ( X ) => 0 requires notBool X [simplification]
```

Now **Kontrol** will attempt to simplify `X` to `true` or `false` in order to apply any of these two new rules. Indeed, if we run again our property test with these added rules, the proof will pass.

## Finishing an existing proof

We've identified the missing reasoning link, great! To include our brand new lemmas in the **KEVM** definition (i.e., make them available for **Kontrol** to reason with) we need to rekompile the project, that is, building it again with the `--rekompile` flag (`kontrol build --rekompile --require ${path_to_lemmas} --module-import ${module}`). After this, re-running the proof with the `--reinit` flag will attempt (and succeed) to prove our property from scratch. But what if we don't want to spend more computational resources than necessary to finish the proof from where we left it? That is, what if don't want to explore those branches that we have already explored and know are not problematic? Enter **KCFG** manipulation.

### Node pruning

From the screenshots above we know that the branch starting at node 21 should be pruned by **Kontrol** itself. However, due to the lack of reasoning arguments, it was not able to do so. Now that we've endowed **Kontrol** with the necessary weaponry, all we have to do is remove the reverting node, node 23, and restart the proof _**without**_ the `--reinit` flag to let **Kontrol** resume the proof without the reverting node, and figure out that such an execution branch will never be traversed.&#x20;

To remove the reverting node we can run the following command

```
kontrol remove-node FixedPointMathLibVerification.testMulWad 23
```

Note that `FixedPointMathLibVerification` is the contract where our property test lives.

With the node pruned, our configuration will look like this

<figure><img src="../../.gitbook/assets/Screenshot from 2023-10-19 20-32-28.png" alt=""><figcaption><p>KCFG after removing the reverting node 23</p></figcaption></figure>

#### Resuming execution

After this, we can resume the proof without the `--reinit` flag&#x20;

```
kontrol prove --test FixedPointMathLibVerification.testMulWad
```

Indeed, after allowing **Kontrol** to reason again about the reverting branch, we're greeted with the following message

```
PROOF PASSED: FixedPointMathLibVerification.testMulWad(uint256,uint256):0
```
