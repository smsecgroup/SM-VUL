# Vulnerability
CVE-xxxx-xxxx

## Vendor
Hong Kong Dollar Token (HKDT)

## Vulnerability Type
Variable Shadowing Confusion & Integer Underflow

## Abstract
We found a vulnerability in the smart contract of "HKDT" token.
Because the storage `_totalSupply` is defined in both parent and child contracts, the functions in parent refer `_totalSupply` defined in the parent whereas functions in child refer `_totalSupply` defined in the child. The problem is that because only the `_totalSupply` defined in the child is initialized, the integer underflow occurs if the `_totalSupply` defined in the parent is referred.

## Details
"HKDT" is an Ethereum ERC20 Token contract. 
The total number of transfers submitted to this contract is 6,252, and 1,478 users holding this token.
Moreover, the last transfer date of this contract is 3 hrs 10 minutes ago which indicates that it is actively used by external users.

![](./img/shadow_00_1.png)
  *Figure 1. HKDT Information*

The problem is that the `_totalSupply` storage variables are defined in both parent and child contracts.

```
contract ERC20Basic {
  uint public _totalSupply;
  ..
}

contract BasicToken is Ownable, ERC20Basic {...}

contract BlackList is Ownable, BasicToken {

  function destroyBlackFunds (address _blackListedUser) public onlyOwner {
    require(isBlackListed[_blackListedUser]);
    uint dirtyFunds = balanceOf(_blackListedUser);
    balances[_blackListedUser] = 0;
    _totalSupply -= dirtyFunds;
    DestroyedBlackFunds(_blackListedUser, dirtyFunds);
  }
  ..
}

contract HKDToken is Pausable, StandardToken, BlackList {
  uint public constant _initialSupply = 10 * 10000 * 10000 * (10 ** uint256(decimals));
  uint public _totalSupply = _initialSupply;
  ..
}

```
`HKDToken` inherited the `BlackList` contract, and `BlackList` contract inherited the `ERC20Basic` contract.
There are two difference copies of `_totalSupply` variables, one in the `ERC20Basic` contract and another one in `HKDToken` contract.
The problem is that the `destroyBlackFunds` function in `BlackList` contract refers the `_totalSupply` defined in `ERC20Basic`, but only the `_totalSupply` defined in `HKDToken` is initialized. Hence, integer underflow will occurs when the `destroyBlackFunds` function decreases the `_totalSupply` value.

## Exploit
The below figure shows the result of calling `destroyBlackFunds` function in `BlackList` contract.
As we can see that the `_totalSupply` variable is changed to an extremely big number due to integer underflow.
Note that the value of `_totalSupply` before calling `destroyBlackFunds` function was 0.

  ![](./img/shadow_00_2.png)
  *Figure 2. The Result of calling _totalSupply function*

## Conclusion
If there are same name of the storage variables in both parent and child contract, they are two separated versions of variables. Initializing one does not initialize the other.

## Reference
https://etherscan.io/address/0x508325285114821151a18e148f4299ea09a9ca05

## Discoverer
Sungjae Hwang (sjhwang87@kaist.ac.kr) and Sukyoung Rry (sryu.cs@kaist.ac.kr)
