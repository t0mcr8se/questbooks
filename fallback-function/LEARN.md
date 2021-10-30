## Solidity `fallback` and `recieve` functions

In this quest, we are going to learn about the `fallback()` and the `receive()` functions in solidity, and we're going to learn how they can be useful in our code.

So first we need to know some concepts:

- Smart contracts can recieve tokens and coins just like regular addresses, but we know that a smart contract does not have a private key, so how will we take the assets out of the contract? will it just be stuck on the contract?

    No it will not be stuck if we use the right logic, we can write some functions to prevent tokens and native assets from getting stuck in a smart contract and we willl see how a bit later.
- We are going to use the [Ownable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) smart contract to determine the Owner who can withdraw the funds that are sent to the smart contract

The following code example can receive native assets and ERC-20 tokens and the owner of the contract can withdraw them.

```javascript
// SPDX-Licence-Identifier: 
pragma solidity ^0.8.0;

import './Ownable.sol';

interface IERC20 {
    function transfer(address recipient, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Receiver is Ownable{
    event Thanks(address sender, uint amount);

    fallback() external payable{
        // This function gets triggered when someone sends a transaction to the contract address and the _calldata does not match any of the function signatures we have in our code
        // Some contracts would return the funds to the sender because the trasnaction is likely done by mistake
        emit Thanks(msg.sender, msg.value);
    }

    receive() external payable{
        // this function is triggered when the _calldata is empty
        emit Thanks(msg.sender, msg.value);
    }

    function withdrawEther() external onlyOwner{
        payable(msg.sender).transfer(address(this).balance);
    }

    function withdrawERC20(address asset) external onlyOwner{
        uint tokenBalance = IERC20(asset).balanceOf(address(this));
        IERC20(asset).transfer(msg.sender, tokenBalance);
    }
}

```

In this contract, we can receive native and ERC-20 assets and withdraw them.

We can also inherit this contract into other contracts to receive and withdraw assets without writing the required logic.

## Fallback:
### Before solidity 0.6.x:
In versions of Solidity before 0.6.x, developers typically used the fallback function to handle logic in two scenarios:
- contract received ether and no data.
- contract received data but no function matched the function called

The main use case of the pre-0.6.x fallback function is to receive ether and react to it, a typical pattern used by token-style contracts to reject transfers, emit events or forward the ether. 

The function executes when a contract is called without any data e.g. via .send() or .transfer() functions. 

The 0.5.x syntax is:

```javascript
pragma solidity ^0.5.0;
contract Receiver {
    function() external payable {
        // React to receiving ether
    }
}
```

The function executes when a contract is called without any data e.g. via .send() or .transfer() functions.

The `fallback()` function is a function that gets executed if none of the other functions match the function identifier.

Also, in our scenario (pre-0.6.x), it executes when `_calldata` is empty.

We can implement any logic inside the function, we can also return the funds to the `msg.sender`, because if we do nothing to the `msg.value` it will be added to our ontract balance.

Example: 
```javascript
function() external payable{
    payable(msg.sender).transfer(address(this).balance);
}
```

This function would resend the funds back to the msg.sender, but it only works on native assets.


A very popular use case of the fallback function is the [DelegateProxy](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies) pattern for implementing upgradable contracts.

We will be talking about this in later tutorials.

### After solidity 0.6.x:
The logic for handling the two scenarios mentioned above had to be split to avoid potential security problems, and that's why the `receive()` function was proposed in solidity 0.6.0 

and the syntax for the fallback function was changed to the following:
```javascript
pragma solidity ^0.6.0;
contract Receiver {
    fallback() external payable {
        // React to receiving ether
    }
}
```

## Receive:
The `receive()` function is a function that get executed if the `_calldata` of a transaction was empty.

We can also implement any logic we want, inside this function. For example; let's say our contract has multiple owners, and by sending native assets to the contract we want to distribute these assets among the owners;

we can do the following;

```javascript
address[] public owners;
event ThankYou(address _sender, uint _amount);

receive() external payable{
    require(msg.value > 0);
    emit ThankYou(msg.sender, msg.value)
    uint amount = msg.value / owners;
    for(uint i=0 ; i<owners.length ; i++){
        payable(owners[i]).transfer(amount);
    }
}
```

The previous function will distribute the assets sent to the contract among the owners.

## Withdraw:
- `withdrawEther()`: this function allows the contract owner to withdraw the ether balance in the contract.

- `withdrawERC20()`: this function allows the contract owner to transfer the ERC20 in the contract to their address.

> More withdraw functions with custom logic can be added also.



