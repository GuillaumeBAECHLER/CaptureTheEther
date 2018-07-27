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

