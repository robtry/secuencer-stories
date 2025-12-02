
# 28 - 11 - 2025

- Please consider that I am new to Arbitrum and all its ecosystem. I could be wrong in some points.
- Its seems that `nitro/go-ethereum` is a fork of geth, (most popular eth client).
- It is constantly being updated to keep up with the latest geth changes.
- The reason for this is that Arbitrum needs to be EVM compatible, so it can run smart contracts and dapps built for eth.
- They are not a fork of ethereum, but a fork of geth.
- They should not modify EVM, Data formats, RPCs, etc. So they can be compatible with eth ecosystem. But they changes
the consensus layer, block production and extra compilers (later).
- Looks like they use `hooks` to modify the behavior of geth without changing its core.
- Without Ethereum L1 arbitrum would be a simple database with EVM.
- Regarding to `go-eth` wouldn't be needed if arb would not implement EVMs, the security could be the same (publishing to eth l1) just by implementing the consensus layer and block production.
- When you join to this crypto world you heard a lot about decentralization. And when you look at arbitrum 
you face a centralized sequencer. This is because the sequencer can not steal funds, censor transactions or invent txs (later).
- The security of arb comes inherently from eth, so even if the sequencer is centralized, the funds are safe.
- Well if the sequencer is "centralized" why do we trust it? or how does arbitrum ensure 100% uptime? (later).


# 27 - 11 - 2025

Ive bee reading about Arbritum and its ecosystem.
So far and basically.

- Arbitrum nitro its the core that Arbitrum one, nova and orbit are based on.
- They are not open source, so if you want to use nitro in eth as L2 you will need to pay for a license [AEP](https://docs.arbitrum.foundation/aep/ArbitrumExpansionProgramTerms.pdf).

- Arbitrum one is the optmistic rollup L2 for general purpose use.
- Arbitrum nova is anytrust faster but not based on eth, so its faster, cheaper but less secure.
- Arbitrum orbit is an L3 that allows you to build your own L2 on top of arbitrum one or nova.

- Ironically, Arbitrum orbit (L3) allows you to modify nitro (the core) without a license. But you have to use Arb as parent chain [Orbit](https://docs.arbitrum.io/launch-arbitrum-chain/faq-troubleshooting/troubleshooting-building-arbitrum-chain).
- This repo uses git [submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) which I have mixed feelings about it. On one hand it allows you to keep your dependencies in check and versioned. On the other hand, it makes the repo more complex to manage and understand.

