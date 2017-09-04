The bitshares blockchain is big and graphene technology stores all the data into RAM at chain replay. 
Most of the time a full node with everything loaded it is not needed and expensive due to the amount of memory the machine need to have available. RAM usage can be reduced significantly by using this `witness_node` executable options.

Here are the 4 new options you can use to reduce RAM:

```
programs/witness_node/witness_node --help
...
  --plugins arg                         Space-separated list of plugins to 
                                        activate
...

Options for plugin account_history:
  --track-account arg                   Account ID to track history for (may 
                                        specify multiple times)
  --partial-operations arg              Keep only those operations in memory 
                                        that are related to account history 
                                        tracking
  --max-ops-per-account arg             Maximum number of operations per 
                                        account will be kept in memory
...
```

/* Using develop branch to have all the last options, this will be removed after release:
```
git checkout -t origin/develop
```
*/

### --plugins

Allows to run only the plugins you want. 

By default, if the plugin parameter is not present in the command line startup the node will start with the default plugins loaded, this are:

- witness
- account_history
- market_history

You can launch a node only with the witness plugin activated like the following if you are after just validating blocks:

`
programs/witness_node/witness_node --data-dir data/my-blockprod --rpc-endpoint "127.0.0.1:8090" --plugins witness
`

### --track-account

Allows to track only the history of selected accounts.

Suppose you have an application wallet or something that it is only interested on the history of 1 account or just a few accounts while no need to spend the memory in the huge amount of account history from the rest of the network. You will still want to do transfers and everything normal, it is just the account history what will not be available.

In order to track the history only for just a one account you may start the node as:

`
programs/witness_node/witness_node --data-dir data/my-blockprod --rpc-endpoint "127.0.0.1:8090" --track-account \""1.2.282"\"
`

To track multiple accounts:

`
programs/witness_node/witness_node --data-dir data/my-blockprod --rpc-endpoint "127.0.0.1:8090" --track-account \""1.2.282"\" \""1.2.24484"\" \""1.2.2058"\"
`

### --max-ops-per-account

Add a number of operations the node will store for each account in the network.

We found out that most of the blockchain size is actually millions of orders automated bot accounts make in the system. The market making this accounts do, very needed for the DEX liquidity is of little value for most nodes.

This parameter allows to go deleting the older operation history from all the accounts, balances and everything will still be fine, please remember this is only deleting history.

By limiting the number of ops per account to 1000 the blockchian decrease in size is more than notorious and will allow you to run nodes in reduced memory machines, can run with 4-5 gigs by using combinations around this option.

Reduce the number of operations for each account that the node will save in the blockchain to 100 by starting with:

`
programs/witness_node/witness_node --data-dir data/my-blockprod --rpc-endpoint "127.0.0.1:8090" --max-ops-per-account 100
`

### --partial-operations 

Remove operation history objects.

Bitshares stores operations in 2 different objects, the 2.9.X and the 1.11.X. they are not exactly the same but if you are removing ops with `--track-account` or `--max-ops-per-account` it makes sense that you also add this option to reduce memory usage even more.

`programs/witness_node/witness_node --data-dir data/my-blockprod --rpc-endpoint "127.0.0.1:8090" --max-ops-per-account 100 --partial-operations true`

### combinations

Combinations that make sense are all valid and can be used to suit your needs.

I personally start my nodes with 1000 ops per account and parttial operations:

`programs/witness_node/witness_node --data-dir data/my-blockprod --rpc-endpoint "127.0.0.1:8090" --max-ops-per-account 1000 --partial-operations true`

This will allow me to run the node with less than 5 gigs(4.820492G):

```
ffffffffff600000      4K r-x--   [ anon ]
 total          4820492K
root@alfredo:~# pmap 28685
```

### special notes

- All the described options are currently in the `develop` branch of bitshares core. all this will be merged to `master` in the next release
- A new option could be `untrack accounts`. we could identify the biggers and run a node with the account history of bots out.