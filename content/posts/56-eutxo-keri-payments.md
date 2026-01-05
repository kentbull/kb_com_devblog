+++
draft = true
title = "Scaling a programmable payments network with EUTxO and KERI"
slug = "eutxo-keri-payments"
date = "2026-01-04"

[taxonomies]
tags=["keri", "blockchain", "cardano", "eutxo"]

[extra]
comment = true
+++

# Scaling a programmable payments network with EUTxO and KERI


Overview

Why EUTxO and not EVM?

Why KERI?

Getting all of the EVM ecosystem to have some way would be really cool though the devil is in the details. The account based model is such a big constraint with the EVM that we aren’t getting any scalability or cost benefits for the payment transaction volumes we’re targeting. For increasing the programmability payments supported featureset then yes, I could see EVM compatibility as extending the capabilities of the DCMS. There are a number of ways to approach something similar to this. We should bring in an EVM expert to discuss potential paths. Fabric has work here though Stellar does not.

There are projects in the Hyperledger Fabric ecosystem such as [Besu](https://www.hyperledger.org/use/besu)  to bring EVM compatibility for smart contracts to Fabric. Whether that means all Ethereum projects would be linkable to the DCMS or just a subset remains to be determined with further research. Stellar smart contracts are not EVM compatible and bridges with wrapped tokens are one way people are connecting things, though bridges are overhead in both technical and financial terms and the bridges are just for token exchange, not for smart contract execution.

One path I see that seems the most practical and promising is implementing a smart contract layer with [EUTxO](https://iohk.io/en/research/library/papers/the-extended-utxo-model/) like Cardano and then using something like [Milkomeda](https://www.milkomeda.com/) to bridge in all EVM functionality. The reason to go through EUTxO  to scaling smart contracts is because EUTxO is scalable. The major drawback of the Ethereum VM is the account-based model and the globally stored state model of the EVM which causes a forced bottleneck in updating balance aggregates, precisely the same bottleneck we’re experiencing in the DCMS right now. Ethereum is just not scalable for money transactions due to the account model.

EUTxO though provides a model for scaling payment transactions as well as smart contracts for rich payment programmability which in my mind is the goal, not necessarily EVM compatibility, at least not right out of the gate.

And to the desire to “add a ton more functionality for less cost vs. any other type of transaction network” I disagree with that because of the inherent constraint of the account based model mentioned above. EVM is not a scaling model and is not a low-cost model, not with the current design of the EVM. Rather, using EUTxO provides a scalable smart contract model that sets us up for a future EUTxO to EVM compatibility layer to be able to bridge to anything in the EVM space while still having a core that is fast and programmable.

### Share this:

- Click to share on X (Opens in new window)
				X
- Click to share on Facebook (Opens in new window)
				Facebook
- 

Categories: [Uncategorized](https://kentbull.com/category/uncategorized/)

Tagged as: [temp-publish](https://kentbull.com/tag/temp-publish/)

### Kent Bull
