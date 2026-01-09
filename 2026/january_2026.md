# 09 - 01 - 2026

- Okay I am going to mirror offchainlabs repos and keep track of the main branches so:

contracts:

```sh
git clone --mirror git@github.com:OffchainLabs/nitro-contracts.git
cd nitro-contracts.git
git remote add bk <>
git push --mirror bk

# to update origin in mirror mode
git fetch origins
git push --mirror bk

# for gittea, since gittea dont like pull requests refs
git remote add gittea <>
git for-each-ref --format='delete %(refname)' refs/pull | git update-ref --stdin
git push gittea --mirror
```

Okay so mirror brings the git objects but not the working tree, fortunately gittea know how to handle bare repos. And this was only needed beacause of those random referenced commits, in my workflow I will only be watching for those main branches. Something like:

```sh
git clone gittea
git remote add arb <> # offchainlabs repo
# for updating
git fetch arb
git checkout main
git merge arb/main # or git rebase arb/main
```

Now I can delete mirror repo `cd .. && rm -rf nitro-contracts.git`. And finally I have all the commits I need. Hmmm not sure if this was an overkill but now I have all the history.

- However I am going to point to another commit, since I have to modify a simple file. But well at least use the same version as offchainlabs.

- Wait a minute, just today the fixed the commit reference to the latest tag in legacy contract hahaha. Okay I need to use nitro in the correct version.

Another useful command is `git tag --sort=-creatordate -n`

```sh
git checkout v3.9.5
git submodule status # to see the commit references
```

Then created my custom branches for those submodules, since they do no have tags in the commits they are using. And actually I am worried about future updates, but well since the change y this repo is minimal (just one file) for now. A cherry pick is going to be enough.

- Apply the same mirror process to `nitro`. And lets continue


# 08 - 01 - 2026

- Definitly faster to make this soft-rebrand changing only the superficial names, and leaving the internal logic as it is. This way we can mantain the codebase more easily with future updates from nitro.
- This soft rebrand was made in `3.9.4` intentionally, arbitrum nitro is now in `3.9.5` so we can test the update process too.
- Perfect, basically a `git rebase v3.9.5`, 0 conflicts. This was an easy one.
- The final list to keep track of custom submodules is going to be: `contracts`, `contracts-legacy` and of course `nitro`. Still thinking if it is work to track the 14 other repos that are referenced in the main nitro repo. 
- Lets make this progressive so for now only 3 repos are going to be tracked as submodules.
- Since I am selfhosted proficient now I am going to have antoher backup, not using gittea, instead bare git repos in a vps that I own. Just in case. But as in the cloud I am going to be backing them up as soon as I need it.
- Something I didn't know is that when you are working with submodules, you have to ways to bring changes or update files: 1. In the repo itself and the pull changes to the main repo. 2. In the main repo, and then go to the submodule folder and push or pull changes from there. Both ways work. Of course you need permissions to to that.

```sh
git clone https://github.com/OffchainLabs/nitro-contracts
cd nitro-contracts
git rename origin arb
git remote add origin <>
git push -u origin main
```

- I think I am going to keep track of offchainlabs repos in `arb` branch pointing to their main branch (later).
- Looks like the commits they are using are not in the main branch at least for legacy contracts. For legacy contracts since march 2025 they do not change the reference. But the commit [5f61a6b0e9799d7ccc0b3811f3893c68a14e43bf](https://github.com/OffchainLabs/nitro-contracts/tree/5f61a6b0e9799d7ccc0b3811f3893c68a14e43bf) has not tag or anything else. There are 13 commits ahead of the last tag and many line changes.


# 07 - 01 - 2026

- Today something came into my mind, and it is, looking in all the files that the AI change and recompile with a panic error given by chaging the name in the smart contracts.
- Probably insted of changing all the files files, I just need to change the basic names in order that the final user feels the change but the internal logic can remain the same, in this way its going to be easier to mantain the code with future updates from nitro. And as far as I dominated the codebase and we implement custom features, we can change the internal logic later.
- Lets make a plan for this. And see how many changes are needed to make the rebrand only superficial and upgradeable friendly.
- I wanted to point out [headscale](https://headscale.net/) as a selfhosted alternative to tailscale, I have it running and finally can say that I overpassed NAT problems,also dicovered that DERP servers exist and tailscale provides free ones.

# 06 - 01 - 2026

- We have a custom gitea repo [git moca](https://git.moca.app/Moca) I am going to migrate the nitro structure there with the submodules approach too.
- I have to learn about `git submodules` but with differente origins
- Noticed that id `42069` and `69420` are alredy taken, checked in [chainlist.org](https://chainlist.org/). No problem we are going to use `6622`. But for testing I am going to be using `69420` in testnet and see how complicated is to send upgrades.
- Second try with the new instructions to replicate the rebranding process. And compare with another agent to see differences.
- While claude works I am going to keep reading docs I think that this is how developers will need to work in the near future.


# 05 - 01 - 2026

Hi everyone, I am back after a long break, the holidays were great! I hadn't took a break in a while so it was much needed. Now I am ready to continue with the Arbitrum project.

- I've seen a lot of updates about Claude: [Claude Continues v2](https://github.com/parcadei/Continuous-Claude-v2) and one of my predictions came true, someone already released [Claude Code Workflow Studio](https://github.com/breaking-brake/cc-wf-studio/) and [Claude How To](https://github.com/luongnv89/claude-howto) the most introductory repo. Another one that I've installed is [Claude HUD](https://github.com/jarrodwatts/claude-hud)
- Time to deep dive into the Arbitrum codebase again. Before this project turns to be "vibecoding arb fork"
- Looks like my claude actually worked better than expected, I will keep using it to help me with the project.
- Today I also create in this repository a CRON git action to check for updates in the arbitrum nitro repo twice a day.
- Selfhosted my first rss feed in order to check arbitrum updates (I really wanted to use rss but didn't find a real use case until now).
- Also requested calude to create a manual for future merges, probably will be nitro updates harder to integrate, beyond the rebrand. Especially if there are changes in the sequencer logic. (later).
- Requested to create a guide for rebranding arb, and lets apply it on my own.

