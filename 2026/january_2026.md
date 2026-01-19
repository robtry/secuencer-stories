
# 16 - 01 - 2026

I haven't used foundry before and looks like a great tool for blockchain development. Continuing with the deployment process:

1. create a set of 4 walleets
1. fund de owner address
1. `make build-reply-env` this will create the `machine.wavm.br` and `module-root.txt` in `target/machines/latest/machine.wavm.br`: the execution machine of L2 compiled to wasm and compressed with brotli.
1. in `contracts` repo `cp scripts/config.example.ts scripts/config.stagenet.ts` and modify the `config.ts` with the new chainid, adresses and values. There are many configs there and it looks like there something called proxy to update those values in the future without redeploying everything.




# 15 - 01 - 2026

Well looks like each GO PROC takes around 24gb of ram, so with 32gb ram computer only 1 proc can run.

I am going to add more swap:

```sh
sudo fallocate -l 16G /swapfile2
sudo chmod 600 /swapfile2
sudo mkswap /swapfile2
sudo swapon /swapfile2
# verify
free -h
```

Okay I am going to try with `GOMAXPROCS=2 make -j1 test-go`

- Today I found the notice that even linus torvalds is using AI haha, he was reluctant at the beginning but finally accepted it. To be honest this make feel better about using AI tools for coding.

- Talking about test it worked with `GOMAXPROCS=2 make -j1 test-go` but pretty slow, I ncrease even more the swap and try with `GOMAXPROCS=4 make -j2 test-go` and lets see how it goes, specially in this RAM crisis period.

Dam even `GOMAXPROCS=2 make -j1 test-go` make my computer unresponsive. Well run this test is going to be painfull until I get a better computer.

Lets continue with smart contracts and necesary configs for rebrand. For now I will not use eth testnet, I am going to go directly to eth mainnet, but it is going to be a test so a testnet on mainnet which I am going to call it a stagenet. And later on I will make teset on testnet.

About the chainid, looks like there are 3 main sources of chain id list:

- [chainlist.org](https://chainlist.org/)
- [ethereum-lists/chains]( https://github.com/ethereum-lists/chains)
- [chainid.network](https://chainid.network/)

This will be important when I deploy to mainnet, in order to make everyone know about the new chain (later).

- Chain id for stagenet: `662201`



# 13 - 01 - 2026

I am still not used to work using submodules, so in order to update a submodule to a new commit, I have to go to the submodule folder, checkout the desired commit, and then go back to the main repo and commit the change.

```sh
cd nitro-testnode
git fetch origin
git checkout <new-commit-or-branch>
cd ..
git add nitro-testnode
git commit -m "Update contracts submodule to new commit"
git push
```

Just to make sure run again `make build` and `docker build -t capu-nitro:latest .` to verify that everything works fine.

It does. Now lets try:

```sh
cd nitro-testnode
./test-node.bash --init --dev
```

And it works fine too. 


```log
INFO [01-14|18:27:34.097] created jwt file                         filename=/home/user/.arbitrum/jwtsecret
INFO [01-14|18:27:34.097] Running Capu nitro node                  revision=development vcs.time=development
```

But looks like it is still using `.arbtrum` in the folder. So lets `rg '\.arbitrum'` to see where this name is used. And try again.

```sh
cd nitro-testnode
./test-node.bash --init --dev --build # to test everything (build included)
```

It takes sometime and trigger fans, but finally in the `nitro-testnode-sequencer` I see the logs:

```log
INFO [01-14|19:38:08.511] created jwt file                         filename=/home/user/.capu/jwtsecret
INFO [01-14|19:38:08.511] Running Capu nitro node                  revision=development vcs.time=development
```

Finally just to make sure everything is working fine, lets run the test suite, again good code organization, everything is in `Makefile`. But first you need to install a lot of dependencies, check [!./guides/01-setup-env.md](./guides/01-setup-env.md)

```sh
make test-all # this will trigger your fans :) and take some time
```

Woops looks like some test break my terminal hahah and I cant not see where does it fails. Lets run one by one to find the problem. Dam I cant belive I am going to need more resources to develop in this project.

`-j <N>` flag is to limit the number of parallel jobs. Lets see if this maybe helpfull.

```sh
make clean 
make -j6 build-node-deps # ok
make -j6 test-go-deps # ok
make -j6 test-go # failed
make -j6 test-rust # pending
make -j4 test-go-challenge
make -j4 test-go-stylus
make -j4 test-gen-proof
```

# 12 - 01 - 2026

- Well today I found that the repo `safe-smart-account` moved from `https://github.com/safe-global/safe-smart-account/` to `https://github.com/safe-fndn/safe-smart-account/`, looks like they moved some repos to a new fundation `https://safe.global/` -> `https://safefoundation.org/` and they referenced also the x account `https://x.com/safefndn`.
- Nvm git does automatic redirects when a repo is moved, so no problem there.
- I didn't explain next steps for rebrand so here it goes:

After the mirror for nitro and nitro-contracts, udpate `.gitmodules` to point to the new repos in gitea.

```sh
git submodule sync
git submodule update --init --recursive
make build # this will trigger your fans :)
```

We can talk about `Makefile` now, around 600 lines, the `build` command triggers a build chain of `yarn`, compile contracts, compile go bindings, brotli, cbingind, rust, jit, go binaries.

I had to make a fix, not sure how do this works for them, I see there prs about `lint_on_build = false` for `foundry.toml` in contacts repo [here](https://github.com/OffchainLabs/nitro-contracts/pull/408/files#diff-bac743fe3d899998e32393af53af8cc7d28c89c3d7fabfa48b01361cf8d42d9b) so eventually it will get merged.

So applied the fix to `contracts`, `contracts-legacy` and `contracts-local`.

Make `git clone` for first time in another computer, and found some problems with submodules, I think I am not being fan of submodules. Well here are the steps:

```sh
git clone --recursive <>
cd nitro
git checkout develop
git submodule sync --recursive
git submodule update --recursive
make build
```

Finally when the build succeeds we can find the binaries in `target/bin/` folder. The principal of course is `./target/bin/nitro --help`.
But there are other useful binaries like: `nitro-val`, `relay`, `deploy`, `da-server`, `el-proxy`, etc.

Before configuring `arbitrum_chain_info.json` for real mainnet o testnet deployment, I am going to test it locally. Lets build now using docker.

```sh
docker build -t capu-nitro:latest .
```

Now its time to modify `nitro-testnode` repo, basically the same mirror commands, and point to my custom nitro.

Change my custom contract version

```sh
# test-node.bash
DEFAULT_NITRO_CONTRACTS_VERSION="custom-branch-or-commit"
```

Just face the problem that docker clones public repo, but for now my fork the repos are private, so I am going to need to copy code instead of cloning.

But `nitro-testnode` its a submodule and it should work alone. Since docker is its own enviroment I need to bridge my ssh key or somehow give access to the repo.

The other option is that we are going to have the limitation that you can only run `nitro-testnode` only from nitro and not in the submodule.
I think I am going to go with this last option for now, since I am going to be the only one making changes to the codebase.

```yml
# nitro-testnode/docker-compose.yml
rollupcreator:
  context: ../  # instead of cloning, use the parent folder
  dockerfile: nitro-testnode/roullupcreator/Dockerfile
```

```Dockerfile
# roullupcreator/Dockerfile

# RUN git clone --no-checkout contracts
# git checkout 
# git submodule update --init --recursive 
COPY contracts ./
```


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

- Apply the same mirror process to `nitro`. And lets continue.


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

