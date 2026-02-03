
```
git clone --mirror git@github.com:OffchainLabs/nitro-contracts.git
cd nitro-contracts.git
git remote add bk <>
git push --mirror bk

# to update origin in mirror mode
git fetch origins
git push --mirror bk

# for gittea, since gittea dont like pull requests refs
git remote add gittea <>.git
git for-each-ref --format='delete %(refname)' refs/pull | git update-ref --stdin
git push gittea --mirror

# git clone .. from gittea
# git remote add other-bk <>
```

- nitro:
- contarcts:
- testnode:
- orbit-setup-script:

