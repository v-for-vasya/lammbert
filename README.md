# LAMMbert Uniswap v4 Hook
### ** Based on the Euler Finance [`tweet`](https://twitter.com/euler_mab/status/1724403149593583745) by Michael Bentley **

[`Bentley's White Paper`](https://github.com/euler-mab/LAMMbert/blob/main/LAMMbert.pdf)

### TLDR ###
Unliked Uniswap v3, Uniswap v4 allows one to create **unique invariants** to offer a reduced price impact and to be able to also provide liquidity in the tails.
A unique red invariant that is between constant sum (x+y=k) and constant product (xy=k) is implemented as a hook based on the work of Michael Bentley utilizing the [Lambert W function](https://en.wikipedia.org/wiki/Lambert_W_function) implemented with the solady fixdepointmath.sol library by [vectorized.eth](https://github.com/Vectorized).

![lambertw](https://github.com/v-for-vasya/lammbert/assets/11951513/4c07ffb9-a085-4dcf-95c0-facbdf3c0342)

1. Default hook from Uniswap Foundation [Counter.sol](src/Counter.sol) demonstrate the `beforeSwap()` hook where the invariant logic is inserted.
2. The invariant curve constant sum template used as our starting point [Counter.t.sol](test/Counter.t.sol) preconfigures the v4 pool manager, test tokens, and test liquidity.

An interesting combo we wanted to implement is to use Euler Vaults inside a `beforeSwap()` for Just-in-Time liquidity provision, then move the gathered fees back into the Euler Vault. 
Potential problem there would be changes in returning the same proportion of assets back as the LP composition changes though and we couldn't figure out how to return the same proportions that we borrowed.

<details>
<summary>Updating to v4:latest</summary>

This template is actively maintained -- you can update the v4 dependencies, scripts, and helpers: 
```bash
git remote add template https://github.com/uniswapfoundation/v4-template
git fetch template
git merge template/main <BRANCH> --allow-unrelated-histories
```

</details>

---

## Set up as outlined by Saucepoints

*requires [foundry](https://book.getfoundry.sh)*

```
forge install
forge test
```

### Local Development (Anvil)

Other than writing unit tests (recommended!), you can only deploy & test hooks on [anvil](https://book.getfoundry.sh/anvil/)

```bash
# start anvil with TSTORE support
# (`foundryup`` to update if cancun is not an option)
anvil --hardfork cancun

# in a new terminal
forge script script/Anvil.s.sol \
    --rpc-url http://localhost:8545 \
    --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 \
    --broadcast
```

<details>
<summary><h3>Testnets</h3></summary>

We ran out of time to test the invariant, only compiled it, but the Goerli deployment is out of sync with the latest v4. **It is recommend to use local testing instead**


```bash
POOL_MANAGER = 0x0
POOL_MODIFY_POSITION_TEST = 0x0
SWAP_ROUTER = 0x0
```

Update the following command with your own private key:

```
forge script script/00_Counter.s.sol \
--rpc-url https://rpc.ankr.com/eth_goerli \
--private-key [your_private_key_on_goerli_here] \
--broadcast
```

### *Deploying your own Tokens For Testing according to Saucepoints*

Because V4 is still in testing mode, most networks don't have liquidity pools live on V4 testnets. We recommend launching your own test tokens and expirementing with them that. We've included in the templace a Mock UNI and Mock USDC contract for easier testing. You can deploy the contracts and when you do you'll have 1 million mock tokens to test with for each contract. See deployment commands below

```
forge create script/mocks/mUNI.sol:MockUNI \
--rpc-url [your_rpc_url_here] \
--private-key [your_private_key_on_goerli_here]
```

```
forge create script/mocks/mUSDC.sol:MockUSDC \
--rpc-url [your_rpc_url_here] \
--private-key [your_private_key_on_goerli_here]
```

</details>

---

<details>
<summary><h2>Troubleshooting</h2></summary>

### *Permission Denied*

When installing dependencies with `forge install`, Github may throw a `Permission Denied` error

Typically caused by missing Github SSH keys, and can be resolved by following the steps [here](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh) 

Or [adding the keys to your ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent), if you have already uploaded SSH keys

### Hook deployment failures

Hook deployment failures are caused by incorrect flags or incorrect salt mining

1. Verify the flags are in agreement:
    * `getHookCalls()` returns the correct flags
    * `flags` provided to `HookMiner.find(...)`
2. Verify salt mining is correct:
    * In **forge test**: the *deploye*r for: `new Hook{salt: salt}(...)` and `HookMiner.find(deployer, ...)` are the same. This will be `address(this)`. If using `vm.prank`, the deployer will be the pranking address
    * In **forge script**: the deployer must be the CREATE2 Proxy: `0x4e59b44847b379578588920cA78FbF26c0B4956C`
        * If anvil does not have the CREATE2 deployer, your foundry may be out of date. You can update it with `foundryup`

</details>

---

