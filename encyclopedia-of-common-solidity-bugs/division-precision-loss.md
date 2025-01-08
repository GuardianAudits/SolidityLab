# ➗ Division Precision Loss

## What are they? 

Solidity use Fixed Point Arithmetic, that mean it doesn't support decimal value.<br>
As a result, any non-integer value is truncated downward.<br>
This characteristic of Solidity can lead to precision loss during numerical operations, especially when division is performed before multiplication, adversely affecting the accuracy of calculations.

```javascript
For Example in Solidity

3 / 2 = 1;

1 / 2 = 0;

```

## Different Kind of Division Precision Loss

Division precision loss can manifest in several ways within Solidity.<br>
This article focuses on the two most prevalent issues:

+ ***Division Before Multiplication***
+  ***Rounding Down To Zero***

### Division Before Multiplication

Solidity truncates any non-integer result to the nearest lower integer.<br>
If a division occurs before a multiplication, the operation may result in precision loss due to truncation.

```javascript
For Example

The expected result is 55.
Solidity make the first calculation 11 / 2 = 5 due to trucation
Then proceed to multiplication.

uint a = 11;
uint b = 2;
uint c = 10;

a / b * c = 50 instead of 55
```

This is a common rule to follow: 
### ***"Always Multiply Before Dividing"***



Although many developers follow this rule, "Hidden Precision Loss" can still occur, resulting from complex calculations across different functions or contracts.<br>
These scenarios are trickier to identify but pose a significant risk if overlooked.

Let's take an example of the USSD Contest on C4: 

```javascript

function rebalance() override public {
      uint256 ownval = getOwnValuation();
      (uint256 USSDamount, uint256 DAIamount) = getSupplyProportion();
      if (ownval < 1e6 - threshold) {
        // @audit amountToBuy is the parameter of this call
        BuyUSSDSellCollateral((USSDamount - DAIamount / 1e12)/2);
      }
}

```

At first glance, the calculation appears correct, let's take a look at the `BuyUSSDSellCollateral` function <br>
```javascript

function BuyUSSDSellCollateral(uint256 amountToBuy) internal {
  CollateralInfo[] memory collateral = IUSSD(USSD).collateralList();
  uint amountToBuyLeftUSD = amountToBuy * 1e12;
  ...
  ...

```
But the `BuyUSSDSellCollateral` function multiplies the input by 1e12, leading to potential precision loss.


So it will first do the calculation inside the parenthesis `(USSDamount - DAIamount / 1e12)/2`
Then call the 'BuyUSSDSellCollateral' function and multiply the result by 1e12.<br>
But mutiply a number that might has been round down by 1 Trillion seems not to be a good idea.

```javascript
If we adjust our example above and change c to 1e12,
The expected result is 5.5e12. However:

uint a = 11;
uint b = 2;
uint c = 1e12;

a / b * c = 5e12

5.5e12 - 5e12 = 0.5e12;

The difference is 0.5e12, indicating a half-a-trillion error. 🤯
```

[Source of findind here](https://solodit.xyz/issues/m-8-buyussdsellcollateral-always-sells-0-amount-if-need-to-sell-part-of-collateral-sherlock-none-ussd-autonomous-secure-dollar-git) 

### Rounding Down To Zero

In Solidity, due to the same feature, if the Numerator is Lower that the Denominator, the result will be 0 <br>
In regular math:<br>

If 
$A < B$ with $A, B > 0$

Then 
$\frac{A}{B} < 1$

So here, the common rule is:
### ***"Always make sure that the Numerator is greater than the Denominator"***

Here's an example in the Cooler Contest on Sherlock: 

```javascript

function errorRepay(uint repaid) external {
    console.log("PrecisionLoss.errorRepay()");
    // If repaid small enough, decollateralized will round down to 0,
    // allowing loan to be repaid without changing collateral
    uint decollateralized = loanCollateral * repaid / loanAmount;

    loanAmount     -= repaid;
    loanCollateral -= decollateralized;
}

```

If `loanCollaterral * repaid` < `loanAmount` -> `decollateralized == 0`
That's bad, we don't want that to happen.<br>
It can be tricky sometime to always make sure of that rule.
That how Security Reasearchers came up with a sanity check. 

```diff
+  require(decollateralized != 0, "Round down to zero");
```

[Source of findind here](https://solodit.xyz/issues/m-3-repaying-loans-with-small-amounts-of-debt-tokens-can-lead-to-underflowing-in-the-roll-function-sherlock-cooler-cooler-git) 


## Conclusion

While division rounding errors might seem minor, they can lead to significant fund risks if overlooked.<br>
This overview only scratches the surface of common division rounding errors in Solidity.
Researcher are encouraged to delve deeper into the subject to understand and mitigate potential precision losses in their audit.



