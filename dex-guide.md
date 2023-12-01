# 🚩 Challenge 4: ⚖️ Build a DEX

![readme-4](https://github.com/scaffold-eth/se-2-challenges/assets/55535804/a4807ee8-555a-4466-8216-0d91e0e76c33)

This challenge will help you build/understand a simple decentralized exchange, with one token-pair (ERC20 BALLOONS ($BAL) and ETH). This repo is an updated version of the [original tutorial](https://medium.com/@austin_48503/%EF%B8%8F-minimum-viable-exchange-d84f30bd0c90) and challenge repos before it. Please read the intro for a background on what we are building first!

🌟 The final deliverable is an app that allows users to seamlessly trade ERC20 BALLOONS ($BAL) with ETH in a decentralized manner. Users will be able to connect their wallets, view their token balances, and buy or sell their tokens according to a price formula!
Deploy your contracts to a testnet then build and upload your app to a public web server. Submit the url on [SpeedRunEthereum.com](https://speedrunethereum.com)!

There is also a 🎥 [Youtube video](https://www.youtube.com/watch?v=eP5w6Ger1EQ) that may help you understand the concepts covered within this challenge too:

💬 Meet other builders working on this challenge and get help in the [Challenge 4 Telegram](https://t.me/+_NeUIJ664Tc1MzIx)

## Checkpoint 0: 📦 Environment 📚

Before you begin, you need to install the following tools:

- [Node (v18 LTS)](https://nodejs.org/en/download/)
- Yarn ([v1](https://classic.yarnpkg.com/en/docs/install/) or [v2+](https://yarnpkg.com/getting-started/install))
- [Git](https://git-scm.com/downloads)

Then download the challenge to your computer and install dependencies by running:

```sh
git clone https://github.com/scaffold-eth/se-2-challenges.git challenge-4-dex
cd challenge-4-dex
git checkout challenge-4-dex
yarn install
```

> in the same terminal, start your local network (a blockchain emulator in your computer):

```sh
yarn chain
```

> in a second terminal window, 🛰 deploy your contract (locally):

```sh
cd challenge-4-dex
yarn deploy
```

> in a third terminal window, start your 📱 frontend:

```sh
cd challenge-4-dex
yarn start
```

📱 Open http://localhost:3000 to see the app.

> 👩‍💻 Rerun `yarn deploy` whenever you want to deploy new contracts to the frontend. If you haven't made any contract changes, you can run `yarn deploy --reset` for a completely fresh deploy.

## Checkpoint 1: 🔭 The Structure 📺

Navigate to the `Debug Contracts` tab, you should see two smart contracts displayed called `DEX` and `Balloons`.

`packages/hardhat/contracts/Balloons.sol` is just an example ERC20 contract that mints 1000 $BAL to whatever address deploys it.

`packages/hardhat/contracts/DEX.sol` is what we will build in this challenge and you can see it starts instantiating a token (ERC20 interface) that we set in the constructor (on deploy).

> Below is what your front-end will look like with no implementation code within your smart contracts yet. The buttons will likely break because there are no functions tied to them yet!

![ch-4-main](https://github.com/scaffold-eth/se-2-challenges/assets/59885513/930ec64c-d185-4a33-8941-43f44a611231)

> 🎉 You've made it this far in Scaffold-Eth Challenges 👏🏼 . As things get more complex, it might be good to review the design requirements of the challenge first!  
> Check out the empty `DEX.sol` file to see aspects of each function. If you can explain how each function will work with one another, that's great! 😎

> 🚨 🚨 🦖 **The code blobs within the toggles are some examples of what you can use, but try writing the implementation code for the functions first!**

---


## Checkpoint 2: Reserves ⚖️

We want to create an automatic market where our contract will hold reserves of both ETH and 🎈 Balloons. These reserves will provide liquidity that allows anyone to swap between the assets.

We want to add two new global variables so the whole contract has access to these values.
The variables will be for `totalLiquidity` and `liquidity`, add them to `DEX.sol`:

<details markdown='1'><summary>👩🏽‍🏫 Socratic Guide</summary>

1. What type should `totalLiquidity` be? We will update its value later so no need to assign it a value.
2. We want the contract to keep track of different people's (addresses) `liquidity` (numerical value), what data structure should we use? Perhaps a key value store?

</details>

<details markdown='1'><summary>👩🏽‍🏫 Solution Code</summary>

```
uint256 public totalLiquidity;
mapping (address => uint256) public liquidity;
```

</details>

These variables track the total liquidity, but also by individual addresses too.
Now, let's create an `init()` function in `DEX.sol` that is payable and assign the value sent to our contract to our global variables we just defined.

<details markdown='1'><summary> 👨🏻‍🏫 Socratic Guide</summary>
1. Let's first require the contract has no totalLiquidity.
2. When our contract has tokens, how would we assign the contract's balance to `totalLiquidity`?
3. We are the ones who sent the liquidity, how would we assign ourselves (as individuals) the liquidity we just provided? How much liquidity have we provided? The `totalLiquidity`? Just half? 
4. Now that we set up where our tokens are going once they are sent, what is the last step to make sure our contract has tokens? Should this last step be 'require'd, or would it be no big deal if our DEX had 0 tokens 😧?
</details>

<details markdown='1'><summary> 👨🏻‍🏫 Solution Code</summary>

```
    function init(uint256 tokens) public payable returns (uint256) {
        require(totalLiquidity == 0, "DEX: init - already has liquidity");
        totalLiquidity = address(this).balance;
        liquidity[msg.sender] = totalLiquidity;
        require(token.transferFrom(msg.sender, address(this), tokens), "DEX: init - transfer did not transact");
        return totalLiquidity;
    }
```

</details>

Calling `init()` will load our contract up with both ETH and 🎈 Balloons.

We can see that the DEX starts empty. We want to be able to call `init()` to start it off with liquidity, but we don’t have any funds or tokens yet. Add some ETH to your local account using the faucet and then find the `00_deploy_your_contract.ts` file. Find and uncomment the lines below and add your front-end address:

```
  // // paste in your front-end address here to get 10 balloons on deploy:
  // await balloons.transfer(
  //   "YOUR_FRONTEND_ADDRESS",
  //   "" + 10 * 10 ** 18
  // );
```

> Run `yarn deploy`.

The front end should show you that you have balloon tokens. We can’t just call `init()` yet because the DEX contract isn’t allowed to transfer ERC20 tokens from our account.

First, we have to call `approve()` on the Balloons contract, approving the DEX contract address to take some amount of tokens.

![balloons-dex-tab](https://github.com/scaffold-eth/se-2-challenges/assets/55535804/710f5c9a-d898-4012-9014-4c46f1de015f)

> 🤓 Copy and paste the DEX address to the _Address Spender_ and then set the amount to 5.  
> You can confirm this worked using the `allowance()` function in `Debug Contracts` tab using your local account address as the owner and the DEX contract address as the spender.

Now we are ready to call `init()` on the DEX, using `Debug Contracts` tab. We will tell it to take 5 of our tokens and send 0.01 ETH with the transaction. Remember in the `Debug Contracts` tab we are calling the functions directly which means we have to convert to wei, so don't forget to multiply those values by 10¹⁸!

![multiply-wei](https://github.com/scaffold-eth/se-2-challenges/assets/55535804/531cab0b-2b37-4489-88c3-d36c0755d2d1)

In the `DEX` tab, to simplify user interactions, we run the conversion (_tokenAmount_ \* 10¹⁸) in the code, so they just have to input the token amount they want to swap or deposit/withdraw.

You can see the DEX contract's value update and you can check the DEX token balance using the `balanceOf` function on the Balloons UI from `DEX` tab.

This works pretty well, but it will be a lot easier if we just call the `init()` function as we deploy the contract. In the `00_deploy_your_contract.ts` script try uncommenting the init section so our DEX will start with 5 ETH and 5 Balloons of liquidity:

```
  // // uncomment to init DEX on deploy:
  // console.log(
  //   "Approving DEX (" + dex.address + ") to take Balloons from main account..."
  // );
  // // If you are going to the testnet make sure your deployer account has enough ETH
  // await balloons.approve(dex.address, ethers.utils.parseEther("100"));
  // console.log("INIT exchange...");
  // await dex.init(ethers.utils.parseEther("5"), {
  //   value: ethers.utils.parseEther("5"),
  //   gasLimit: 200000,
  // });
```

Now when we `yarn deploy --reset` then our contract should be initialized as soon as it deploys and we should have equal reserves of ETH and tokens.

### 🥅 Goals / Checks

- [ ] 🎈 In the DEX tab is your contract showing 5 ETH and 5 Balloons of liquidity?
- [ ] ⚠ If you are planning to submit the challenge make sure to implement the `getLiquidity` getter function in `DEX.sol`

---

## ⛳️ **Checkpoint 3: Price** 🤑

This section is directly from the [original tutorial](https://medium.com/@austin_48503/%EF%B8%8F-minimum-viable-exchange-d84f30bd0c90) "Price" section. It outlines the general details of the DEX's pricing model.

If you need some more clarity on how the price in a pool is calculated, [this video](https://youtu.be/IL7cRj5vzEU) by Smart Contract Programmer has a more in-depth explanation.

Now that our contract holds reserves of both ETH and tokens, we want to use a simple formula to determine the exchange rate between the two.
Let’s start with the formula `x * y = k` where `x` and `y` are the reserves:

```
(amount of ETH in DEX ) * ( amount of tokens in DEX ) = k
```

The `k` is called an invariant because it doesn’t change during trades. (The `k` only changes as liquidity is added.) If we plot this formula, we’ll get a curve that looks something like:

![image](https://user-images.githubusercontent.com/12072395/205343533-7e3a2cfe-8329-42af-a35d-6352a12bf61e.png)

> 💡 We are just swapping one asset for another, the “price” is basically how much of the resulting output asset you will get if you put in a certain amount of the input asset.

🤔 OH! A market based on a curve like this will always have liquidity, but as the ratio becomes more and more unbalanced, you will get less and less of the less-liquid asset from the same trade amount. Again, if the smart contract has too much ETH and not enough $BAL tokens, the price to swap $BAL tokens to ETH should be more desirable.

When we call `init()` we passed in ETH and $BAL tokens at a ratio of 1:1. As the reserves of one asset changes, the other asset must also change inversely in order to maintain the constant product formula (invariant `k`).

Now, try to edit your `DEX.sol` smart contract and bring in a price function!

The price function should take in the reserves of `xReserves`, `yReserves`, and some input (`xInput`) to calculate what the exchange rate of $BAL to ETH or vice versa.
Don't forget about trading fees! These fees are important to reward and incentive liquidity providers. Let's make the trading fees 0.3% and remember that there are no floats (decimals) in Solidity, only whole numbers!

<details markdown='1'><summary>👩🏽‍🏫 Socratic Guide</summary>
1. 
</details>

<details markdown='1'><summary>👩🏽‍🏫 Solution Code</summary>

```

    function price(
        uint256 xInput,
        uint256 xReserves,
        uint256 yReserves
    ) public pure returns (uint256 yOutput) {
        uint256 xInputWithFee = xInput * 997;
        uint256 numerator = xInputWithFee * yReserves;
        uint256 denominator = (xReserves * 1000) + xInputWithFee;
        return (numerator / denominator);
    }

```

</details>

We use the ratio of the input vs output reserve to calculate the price to swap either asset for the other. Let’s deploy this and poke around:

```
yarn run deploy
```

Let’s say we have 1 million ETH and 1 million tokens, if we put this into our price formula and ask it the price of 1000 ETH it will be an almost 1:1 ratio:

![price-example-1](https://github.com/scaffold-eth/se-2-challenges/assets/55535804/e2d725cc-91f3-454d-902f-b39e4b51f5e2)

If we put in 1000 ETH we will receive 996 tokens. If we’re paying a 0.3% fee it should be 997 if everything was perfect. BUT, there is a tiny bit of slippage as our contract moves away from the original ratio. Let’s dig in more to really understand what is going on here.
Let’s say there is 5 million ETH and only 1 million tokens. Then, we want to put 1000 tokens in. That means we should receive about 5000 ETH:

![price-example-2](https://github.com/scaffold-eth/se-2-challenges/assets/55535804/349db3d8-e39e-4c94-8026-e01da2cefb8e)

Finally, let’s say the ratio is the same but we want to swap 100,000 tokens instead of just 1000. We’ll notice that the amount of slippage is much bigger. Instead of 498,000 back we will only get 453,305 because we are making such a big dent in the reserves.

![price-example-3](https://github.com/scaffold-eth/se-2-challenges/assets/55535804/f479d7cd-0e04-4aa7-aa52-cef30d747af3)

❗️ The contract automatically adjusts the price as the ratio of reserves shifts away from the equilibrium. It’s called an 🤖 _Automated Market Maker (AMM)._

### 🥅 Goals / Checks

- [ ] 🤔 Do you understand how the x\*y=k price curve actually works? Write down a clear explanation for yourself and derive the formula for price. You might have to shake off some old algebra skills!
- [ ] 💃 You should be able to go through the price section of this tutorial with the sample numbers and generate the same outputChange variable.

> 💡 _Hints:_ For more information on calculating the Output Reserve, read the Brief Revisit of Uniswap V2 in [this article](https://hackernoon.com/formulas-of-uniswap-a-deep-dive).

> 💡💡 _More Hints:_ Also, don't forget to think about how to implement the trading fee. Solidity doesn't allow for decimals, so one way that contracts are written to implement percentage is using whole uints (997 and 1000) as numerator and denominator factors, respectively.

---
