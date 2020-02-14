# Vulnerability
CVE-XXX

## Vendor
ETA Token

## Vulnerability Type
Typo in Constructor

## Abstract
We found a vulnerability in smart contract of "ETA" token. Because there is a typo in constructor, the attacker can obtain ETA tokens for free.

## Details
'ETA' is a Ethereum ERC20 Token contract. The total number of transactions submit to this contract is 1,622, and 117 users holding this token.

![](./img/ETA_1.PNG)
  *Figure 1. ETA Token Information*

In the constructor of the 'ETA' contract, all the initial tokens are assigned to constructor caller. 

The problem is that the constructor of the ETA token becomes normal `public` function which can be called by external attackers due to the typo in constructor.

By just calling the typo constructor, attacker can obtain tokens for free. 
Note that attacker can control the parameters of the constructor and therefore, he can obtain as many tokens as he wants.

```
contract TokenETA {
...
function ETAToken(
    uint256 initialSupply,
    string tokenName,
    string tokenSymbol
) public {
    totalSupply = initialSupply * 10 ** uint256(decimals);  // Update total supply with the decimal amount
    balanceOf[msg.sender] = totalSupply;                // Give the creator all initial tokens
    name = tokenName;                                   // Set the name for display purposes
    symbol = tokenSymbol;                               // Set the symbol for display purposes
}
```


## Exploit

  Below figure is the result of `ETAToken` function. We can obtain Tokens for free easily.

  ![](./img/ETA_2.PNG)
  *Figure 2. The Result of ETAToken() function*

## Conclusion
The constructor name must be same as contract name or `constructor` keyword should be used for defining constructor.

## Reference
https://etherscan.io/token/0xa00149339410f2936722c4c6f88ae5ad87af6a87

## Discoverer
Sungjae Hwang (sjhwang87@kaist.ac.kr) and Sukyoung Rry (sryu.cs@kaist.ac.kr)