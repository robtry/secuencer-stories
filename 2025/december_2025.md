# 5 - 12 - 2025

- **Native Token**: A simple token that exists only in L2. ERC20. Not gateway needed.
- **StandardERC20 gateway**: Token exists in L1 but you want to use the benefits of L2. It gets locked in l1 and minted in l2.
- **Generic custom gateway**: Token exists in l1, but you want more powers or change characteristics in l2. You can customize the behavior of the token in l2. you do not mint in l2.
- **Custom gateway**: L2 and L1 tokens can behave differently.
- **WETH gateway**: Wrap and unwrap eth between l1 and l2. ETH is not erc20 so it needs a special gateway.
- **Token gateway**: cotracs for the token bridges between l1 and l2 communication.
- **Reverse Token Gateway**: when the token borns in l2 and you want to withdraw to l1.


```txt
  ¿Tu token ya existe en L1?
  │
  ├── NO → Token Nativo L2 
  │
  └── SÍ → ¿Es WETH?
           │
           ├── SÍ → WETH Gateway
           │
           └── NO → ¿Necesitas características custom en L2?
                    │
                    ├── NO → Standard ERC20 Gateway
                    │
                    └── SÍ → ¿Necesitas mintear en L2 y retirar a L1?
                             │
                             ├── NO → Generic Custom Gateway
                             │
                             └── SÍ → Custom Gateway 
```

- **Native fee token**: an erc20 can be used to pay fees in l2 instead of the native token (like arb one uses eth).
- **Ink**: the equivalent of gas stylus vms (since operations are cheaper).
- **Outbox**: merkle tree that stores the root of all outgoing messages.
- **RaaS**: Rollup as a Service, allows you to create your own rollup chain using arbitrum nitro as base layer
- **RPblock**: assertion
- **Reorg**: tx on chain that at some point are considerer valid get reverted. fraud proofs do not reorg just parent chain.
- **Retrayable Ticket**: l1 to l2 messsage that can be retried if it fails. 7 days to retry.
- **Retrayable Autoredeem**: l1 to l2 message that is automatically retried until success.
- **Retrayable Redeem**: if the ticket fails you can manually retry it. or can be initiated by the autoredeem.
- **Sequencer**: the entity that can reorder txs. He is different from validators.
- **Sequencer Feed**: offchain service that provides sequencer data
- **Sequencer Inbox**: contracts that holds messages sent by clients. A message can reach the sequencer inbox directly by the sequencer or delay inbox.
- **Delayed Inbox**: contract that hold parent chain messages, that will be included in the sequencer inbox, the inclusion do not depends on the sequencer.
- **Shared sequencer**: multiple rollups share the same sequencer.  Not used in arb one or nova.
- **Soft Confirmation**: when an assertion is confirmed but not yet finalized in parent chain.
- **Speed Limit**: [currently target 7,000,000 gas / second. When computation exceeds this limit, fees rise, via EIP-1559.](https://docs.arbitrum.io/intro/glossary#speed-limit)
- **State transition function**: STF how new block are computed given the input messages.

I love that arbitrum docs always use "a gentle introduction" in their titles.

- Inbox (gateway) -> STF (state transition function) -> Outbox (destination)
- Identical inputs always produce the same outputs (deterministic).


### Arb process

1. **Submit tx**: user sends tx to sequencer inbox (directly or via delay inbox contract (eth)).
- the sequencer can not censor txs.
2. **Ordering**: sequencer orders txs and creates new block.

# 4 - 12 - 2025

- I tried to reverse proxy my isp router so I can access remote without vpn, vps or cloudflare tunnel. But nowadays most isp do not offer public ipv4 anymore. I don't want to fight with ipv6 yet. So I will mix cloudflare tunnel + vps (later). I wouldnt fight with isp again (too much time wasted).

- **Challenge Protocol**: Claims [The Challenge Protocol guarantees that only valid assertions will be confirmed provided that there is at least one honest active validator.](https://docs.arbitrum.io/intro/glossary#challenge-protocol). Can a node challenge the whole network? (later).
- **Confirmation**: Once it is confirmed, goest to parent chain.
- **Dev-Tools Dashboard**: Check this tools, it might helpme to debug my txs (later).
- **Fast Exit / Liquidity Exit**: Since people don't want to wait 7 days to withdraw their fund there are third party lps that allow you to withdraw instantly with a fee. (later).
- **Force-Inclusion**: Means that is hard for the sequencer to censor txs, because users can force the inclusion of their txs by submitting them directly to eth l1.
- **Gas Price Floor**: gas price on an Arbitrum chain; currently 0.1 gwei on Arbitrum One

I haven't dive in gas concepts, here we go:

- Operations in chains costs gas to avoid spam and pay for resources.
- wei is not for weight, its a tribute to Wei Dai, a cryptographer.
- wei = smallest unit of eth (1 eth = 10^18 wei).
- gwei = 10^9 wei (1 gwei = 0.00000001 eth).
- gas price is the amount of wei you are willing to pay per unit of gas (gwei).
- On arb since posting into L1 requires gas, they also need to cover that cost.
- MAX limit gas per block in arb is 32M gas.
- So arb needs a huge amount of txs to cover the l1 posting cost, if the get few txs it would be unsustainable. Or the will need to increase fees. But if that was the case, why does l2 exist?. Or do they cover the gas fee l1 cost?

# 3 - 12 - 2025

- I created a roadmap to follow instead of jumping randomly between topics. Current phase is still exploring and understanding concepts.
- Today Eth is implementing [Fusaka](https://ethereum.org/roadmap/fusaka/) (Fulu + Osaka) upgrade. I have no idea what it is about yet so:
   - "[The upgrade aims to improve network scalability and reduce costs for Layer 2 networks](https://www.coingecko.com/learn/what-is-ethereum-fusaka-upgrade)" ( benefit for me I guess) https://x.com/ethereum/status/1995510105857490957?s=20.
   - This will be a ![hard fork](../GLOSSARY.md#hard-fork), I still do not understand if any version upgrade can be called "hardfork" or only when there is a breaking change or if people disagree with the upgrade and split the chain.
   - PeerDAS (Peer Data Availability Sampling): not need to check all data, just random samples.
   - Passkeys for wallet login (interesting). https://x.com/ethereum/status/1995510110379250104?s=20
- Okay looks goods I have to be careful about eth information and see if with this upgrade something changes in arb too (later).
- They already have a notice for the upgrade https://docs.arbitrum.io/notices/fusaka-upgrade-notice

- Looks like hard forks exits because of software distribution problems, not everyone updates at the same time. Hence to force everyone to update the software, a hard fork is needed. This makes me think that probably smaller chains can be more controlled and avoid hard forks.
- And hardforks apply only if the consensus rules change, otherwise it is just a software update. But I still do not understand why chains with an active community nodes do "hardforks" if they can just update and distribute the new software version as usual (later).

The infortmation I'm gonna write down are already in the docs, but I feel that writing them down helps me to understand and remember better. Also make some modifications to fit my understanding.

- Make sure to understand all the concepts in this [arb glossaty](https://docs.arbitrum.io/intro/glossary) I think its the best starting point. Because it describes roles:
   - **Active Val**: bonded val that make disputables assertions, can challenge assertions but its not the Sequencer.
   - **Arb token bridge**: a set of contracts that allow users to move their tokens (erc-20) between eth and arb.
   - **Arb Nova**: arb nitro using anytrust security model, cheaper and faster but less secure. uses permissioned set of parties (DAC - Data Availability Committee) to ensure data availability.
   - **Arb Classic**: previous version of arb before nitro, not maintained anymore. It used a custom virtual machine (AVM) instead of EVM.
   - **Arb fullnode**: a listener that knows the state of the arb chain and receives remote procedure calls (RPCs) from clients.
   - **Arb nitro**: the core of arb one and nova, open source but needs a license to be used in eth L2.
   - **Arb one**: optimistic rollup L2 for general purpose use, secured by eth.
   - **ArbOS**: Arbitrum's "operating system" that trustlessly handles system-level operations; includes the ability to emulate the EVM. I think its the core of arbitrum is the intemediary between evm and sequencer (later).
   - **Assertion**: a bonded claim made by a val about the state of the arb chain. Can be proposed ( a val submits an assetion ), challenged ( another val disputes the assertion and a fraud proof starts ) or confirmed ( no challenges within the challenge period 6.4 days, assertion is accepted as valid ).
   - **Auction contract**: smart contract that handles bids to be included in the sequencer (like vip priority) (later). Timeboots needs to be enabled.
   - **Autonumous: auctioner**: offchain (to avoid more txs onchain) receives bids, selects the winners and submits them to the auction contract (later).
   - **BLS Signature**: used in any trust model (arb nova) to aggregate signatures from multiple parties (DAC) into a single signature.
   - **BoLD**: [Bounded Liquidity Delay](https://medium.com/offchainlabs/bold-permissionless-validation-for-arbitrum-chains-9934eb5328cc). new challenge protocol to eliminate [delay attack vectors](https://medium.com/offchainlabs/solutions-to-delay-attacks-on-rollups-434f9d05a07a)
   - **Challenge**: when two bonders disagree about the verediction of an assertion

I am gonna pause here I just realized that:

1. You can pay to win priority in the sequencer (like a vip line). This already happens in other chains whit gas fees, but here it looks like its made outside the chain to avoid extra txs and fees. So looks like even if you send txs the sequencer checks if they came from the auction winner (later).
1. This make me ask if arb allows a huge amount of txs, why do we need to bid those? I mean there should be enough space for everyone right? (later).
1. Looks like this is related to MEV (later).

Next revelation:

1. The challenge definition says [the protocol guarantees that an honest party will always win a challenge](https://docs.arbitrum.io/intro/glossary#challenge).
1. If you lose a challenge you loose all your bond. Which is around [3600 eth](https://docs.arbitrum.io/launch-arbitrum-chain/faq-troubleshooting/troubleshooting-building-arbitrum-chain#:~:text=Bonds%20to%20participate%2D%20Depends%20on,ETH%20per%20each%20subsequent%20challenge.)
1. All the bond that is lost goes to ARB and only 1% to the winner of the challenge.
1. Here are my concerns:
   1. There no high incentive to be an honest val, why would I risk 3600 eth?
   1. Probably there is no high incentive because you can trigger false challenges to make money from other bonders mistakes.
   1. The punishment is high what if I make an honest mistake?
   1. If you can always get the a deterministic result why do you need challegesa at all?
   1. What if you have two validators and you challenge yourself? (later).
   1. Can an assetion be challenged multiple times until there is a final verediction or is it just once? (later).
   1. Looks like they have whitelist challengers.
   1. I see now why they are working on BoLD to reduce the attack.

Following with arb glossary:

   - **Bridge**: Every Arbitrum chain includes a bridge to/from its Parent chain.
   - **Bonder**: a validator who deposits bond (eth) to vouch for a assetion. if false the bond is lost.
   - **Underlying chain**: Parent chain
   - **Validator**: a arb fullnode that tracks status of assertions. Can be watch tower, defensive or active.
     1. Active: propose and challenge assertions. Has bond at stake permanently.
     1. Defensive: only challenge invalid assertions. Only bond when challenging.
     1. Watchtower: neber bonds or take actions, only alerts offchain

There is no incentive to be a watchtower or active val. The best role is defensive val because if you bet is because you are sure the assertion is false.
This is fully centralized chain.

# 2 - 12 - 2025

- I'm not used to read whitepapers, I thing I need to improve my skimming skills to find the important parts faster.
- I think I feel ready to read [arbirum's intro](https://docs.arbitrum.io/how-arbitrum-works/inside-arbitrum-nitro) now.
- Also it would be great to read [AIP](https://www.tally.xyz/gov/arbitrum/proposals) (later).
- Friendly reminder to execute `sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv` after ubuntu server instalation using the LVM option.

I think I will need more time to understand the theory before jumping deeper into the code.

# 1 - 12 - 2025

- I won't be working weekends hahah I have a life outside coding :). (At leat for now).

Since Arb is a L2 based on ETH, I need to understand how ETH works first.

- So go-eth also in ethereum is related to the evm execution layer (p2p). And state management ([mempool](../GLOSSARY.md#mempool)), txs, balances.
- Prysm Beacon Node conects to validator, is exposed to internet p2p and syncs the beacon chain and geth. Only knows about block head, next proposer and finality. DO NOT know about txs or balances. (consensus)
- Prysm Validator is the validator sings the blocks and proposes them to the beacon chain (when selected).Using geth.

Of course a good entry point to understand eth is its [Ethereum Whitepaper](https://ethereum.org/en/whitepaper/).
Nowdays it has a lot of content, explaining the ecosystem, they even implemented [quizzes](https://ethereum.org/quizzes/).

- Definitly helpful to understand the big picture https://ethereum.org/whitepaper/#ethereum

I complemented with

- [Ethereum Yellowpaper](https://ethereum.github.io/yellowpaper/paper.pdf) which is more technical (I didnt read all of it).
- [Ethereum Explained Visually](https://youtu.be/QGH8XnUnOps). Which drivesme crazy about how EVM pretend to work.
- Thanks to the previos video, somehow I ended up in [math incompleteness](https://youtu.be/HeQX2HjkcNo) and turing machines (which are concepts I only recall from college).
- I did'nt read but it would be great to check [EIP](https://eips.ethereum.org/)

I feel excited to learn about the EVM, actually talking about lines of code, it is the heaviest part of arbitrum nitro (later).
