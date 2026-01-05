+++
title = "Using the pRVN BEP-20 pToken from pNetwork and OpenDAO"
slug = "using-prvn"
date = "2021-05-19"

[taxonomies]
tags=["guide", "ravencoin", "cryptocurrency"]

[extra]
comment = true
+++

# Using pRVN from OpenDAO

Ravencoin (RVN) is spreading its wings to the Binance Smart Chain as a BEP-20 token. This opens options up to holders of Ravencoin to earn returns from participating in liquidity pools, to gaining credit by collateralizing loans, and to earn returns from minting and staking stablecoins. This is all made possible by the bridging mechanism produced by the pNetwork and OpenDAO teams.

Minting stablecoins with Omnicoin’s DApp is one exciting usage of the pRVN pToken. See the Omnicoin DApp here: https://omnicomp.ocp.finance/#/
Edit: Use v1 of Omnicomp’s DApp: https://v1omnicomp.ocp.finance/#/ pRVN is not yet listed (as of 7/12/21) in the new DApp.

Disclaimer: This is an advanced-level De-Fi article. Use at your own risk! Detailed instructions are provided yet this is still somewhat dense and early in the evolution of this ecosystem. Bear this in mind.

## What is openDAO?
OpenDAO is is a toolkit to connect the real world to DEFI using exchanges, tokenization, multi collateral stablecoins, incentivized farms, liquidity mining, and stock APIs. We are working directly with OpenDAO to facilitate creation of incentivized farms for Ravencoin, the $RVN0 stablecoin, the Ravencoin stablecoin, and to create liquidity pools with token pairings including pRVN.

## What is pNetwork?
pNetwork’s tokens are a solution to enable cross-chain interactions. This includes interaction with non-EVM and EVM chains using pTokens, including pRVN, as 1:1 pegged representations of one pToken to one unit of the original currency. This means 1 pRVN is equivalent to 1 RVN. 

Why use the pRVN pToken over Ravencoin?
DeFi Compatibility. With the BEP-20 compatible tokens created by the pNetwork team you can participate in various return-generating or credit-generating activities including participating in liquidity pools, creating and staking stablecoins for bridges, collateralizing your pRVN to mint stablecoins, and collateralizing your pRVN to loan stablecoins, among other uses.

In the short term using BEP-20 tokens will give you quicker access to the DeFi ecosystem since the ecosystem is already compatible with BEP-20 tokens. Over time, as Ravencoin is further integrated into the DeFi ecosystem, you will be able to use Ravencoin directly. Until then make use of the BEP-20 and ERC-20 tokens as long as you trust the contract administrators like pNetwork who issue those tokens.

How to get the pRVN pToken?
This is known as a peg-in transaction. You are pegging your crypto currency into a smart contract in order to get an equivalent supply of a BEP-20-compatible token which in this case is pRVN.

1. Purchase Ravencoin
2. Link your Metamask account to the pNetwork DApp for Ravencoin (and other coins): https://dapp.ptokens.io/swap?asset=rvn&from=rvn&to=bsc 
3. Generate a deposit address
4. Send your Ravencoin to the deposit address. The OpenDAO bridge and the pNetwork pToken smart contract for pRVN will hold your Ravencoin and in exchange give you and equivalent value of pRVN to spend in the BEP-20 compatible ecosystem.
5. Add your pRVN pToken to your wallet of choice such as MetaMask
Start by opening MetaMask in your browser and swapping the network to Binance Smart Chain (see the upper right hand area).\

   Click “Add Token”
6. Add in the pNetwork Smart Contract address of:

   0x0c80075886d9e64cc33ca701a4ad1c0d8f0bd651

   Copy and paste the above address into the “Token Contract Address” box. The other fields will auto-populate..

   Click “Next”
7. Verify your pRVN from the pNetwork DApp show up:

   Click “Add Tokens”
8. Now you can send your pRVN tokens anywhere including back to the pNetwork DApp for a Peg-out transaction.
You can also spend your pRVN for other purposes such as collateral for minting stablecoins at https://omnicomp.ocp.finance/#/ 

## How to get from pRVN back to Ravencoin?

At some point you may want to convert your pRVN tokens back to RVN. In that case you use your wallet, MetaMask in this case, to send your pRVN back to the pNetwork bridging smart contract to unlock, or un-peg, your original Ravencoin, thus the name peg-out.

1. Go to the pNetwork DApp to swap back from pRVN to RVN
https://dapp.ptokens.io/swap?asset=rvn&from=rvn&to=bsc
2. Click Swap
3. Authorize the transaction through MetaMask and you will then receive your Ravencoin in the wallet at the address you specified.
Note: You must fund this transaction, like ETH Gas, with Binance Smart Chain BNB. This means you need to purchase BNB for Binance Smart Chain and then send enough of it to your MetaMask wallet to fund the transfer. The transfer fee was about four cents USD for my transaction.
4. You can monitor the transaction as it is sent through the pNetwork bridge:
5. Once the transaction is confirmed the Ravencoin should show up in your Ravencoin Wallet

## Using the pRVN BEP-20 pToken

You can mint stablecoins by collateralizing your pRVN with Omnicoin’s DApp here: https://omnicomp.ocp.finance/#/

Those stablecoins, and pRVN itself, can be used as asset pairs in liquidity pools to facilitate swaps between currencies. These liquidity pools are typically incentivized in the beginning in order to achieve critical mass. During that incentivization period you may earn very high rates of return.

This is all very new so use at your own risk!