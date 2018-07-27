# Capture the Ether

> [Capture the Ether](https://capturetheether.com/challenges/) is a game in which you hack Ethereum smart contracts to learn about security.

This game is brought to you by [@smarx](https://twitter.com/smarx).

# Disclaimer

As I am new to Solidity, I might have made some mistakes here.

That could be on the way I dealt with some challenges or on technical terms I (didn't) used.

I strongly encourage you to double check the information below and send an issue / PR if there is anything not fairly accurate or if you find a prettier workaround !

That said, Lets a go ! :fist:

# Challenge Quests !

> Time for a true display of skill !

## Warmup

### Deploy a contract

All you have to do is installing [Metamask](https://metamask.io/), switching to the Ropsten test network and getting some ether from the [Ropsten metamask faucet](https://faucet.metamask.io/).
When you click the `Begin Challenge` button, metamask will be triggered to ask you to send the transaction in order to deploy the contract.

Nailed it :tada:

### Call me

In order to call the function on the deployed contract, I used [Remix](https://remix.ethereum.org/) wich is pretty easy to use.

You just have to create a file where you copy/paste the contract given on Capture the Ether website. Compile it and go to the Run tab, in the second block section you can load a contract from an existing address or deploy it. After pressing the `Begin Challenge` button, you will be displayed an address, this is this one that you have to copy/paste into the `Load contract from Address` field and press the `At Address` button.

Then in the Deployed Contracts section you should be able to call the `callme()` function !

### Choose a nickname

Same thing as above, you just have to pass an argument next to the function call button.
Pretty easy.

## Lotteries

### Guess the number

>I’m thinking of a number. All you have to do is guess it.

```javascript

pragma solidity ^0.4.21;

contract GuessTheNumberChallenge {
    
    uint8 answer = 42;

    function GuessTheNumberChallenge() public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);
        if (n == answer) {
            msg.sender.transfer(2 ether);
        }
    }

}
```

As we can see in this solidity contract, the answer is stored as it in a global variable named answer.

> It's time to D-D-D-D-Duel !

That was really not a big deal to find that the number you were thinking of was 42 ! :sunglasses:

### Guess the secret number

>Putting the answer in the code makes things a little too easy.
>
>This time I’ve only stored the hash of the number. Good luck reversing a cryptographic hash!

```javascript
pragma solidity ^0.4.21;

contract GuessTheSecretNumberChallenge {
    bytes32 answerHash = 0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365;

    function GuessTheSecretNumberChallenge() public payable {
        require(msg.value == 1 ether);
    }
    
    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);

        if (keccak256(n) == answerHash) {
            msg.sender.transfer(2 ether);
        }
    }
}
```

As we can't reverse a cryptographic hash, the only way we can find it is brute forcing it.
Hopefully, it is possible because we only have 256 possibilities ! ( because n is a uint8 so possible values goes from 0 to 255 ).

Keccak256 is an alias for sha3, so we can use the web3 sha3 function to calculate the hash !

Let's do it in our javascript console with a tiny script :

```javascript

const hash = "0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365"

for(i=0;i<256;i++){
  if(web3.sha3(i) == hash){
    console.log(i)
  }
}

```

> No result found !

NO GOD! PLEASE NO!!! What is going on ?

Well... the sha3 function of Solidity is taking arguments in hexadecimal format.

```javascript

const hash = "0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365"

for(i=0;i<256;i++){
  if(web3.sha3(i.toString(16), {encoding: 'hex'}) == hash){
    console.log(i)
  }
}

```

> It's time to D-D-D-D-Duel !

I guess the number is 170 ! :sunglasses:

### Guess the random number