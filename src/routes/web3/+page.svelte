<script>
    import { onMount } from 'svelte';
    import { Web3 } from 'web3';

    let warning = undefined
    let walletnetwork = undefined
    let networkname = "unknown"
    let walletaddr = undefined
    let walletbalance = undefined

    onMount ( () => {
        if (window.ethereum) {
            window.web3 = new Web3(window.ethereum);
        } else {
            warning = "no web3 wallet attached!"
        }
    })

    async function get_wallet_chainid() {
        try {
            window.ethereum.request({ method: 'eth_chainId' }).then(function(v) {
                walletnetwork = v;
                if (walletnetwork === "0x1") { networkname = "ethereum" }
                if (walletnetwork === "0x89") { networkname = "polygon" }
                if (walletnetwork === "0xa4b1") { networkname = "arbitrum" }
                if (walletnetwork === "0xa86a") { networkname = "avalanche" }
            })
            warning = undefined;
        } catch (error) {
            warning = "error while accessing wallet: " + error;
        }
    }
    function reset_warning() { warning = undefined }

    async function wallet_logout() {
        walletbalance = undefined
        walletaddr = undefined
        walletnetwork = undefined
        networkname = "unknown"
        warning = undefined
    }
    async function get_wallet_balance(ev) {
        if (window.web3 && walletaddr != undefined) {
            ev.target.setAttribute("disabled","disabled")
            walletbalance = await window.web3.eth.getBalance(walletaddr);
            warning = undefined
            ev.target.removeAttribute("disabled")
        } else {
            walletbalance = undefined;
        }
    }
    async function get_wallet_addr(ev) {
        if (window.web3) {
            ev.target.setAttribute("disabled","disabled")
            try {
                // get active network in wallet
                get_wallet_chainid()
                // request user's account address - prompts Metamask to login
                const selectedAccount = await window.ethereum
                    .request({
                        method: "eth_requestAccounts",
                    })
                    .then((accs) => accs[0])
                    .catch(() => {
                        throw Error("Please select an account in your wallet");
                    });

                walletaddr = selectedAccount;
                warning = undefined

            } catch (error) {
                warning = "error while accessing wallet: " + error;
            }
            ev.target.removeAttribute("disabled")
        } else {
            warning = "wallet not found";
        }
    }    
</script>

<svelte:head>
        <title>Wallet</title>
        <meta name="description" content="wallet" />
        <script src="web3.min.js" />
</svelte:head>

<h1>Web3 wallet login</h1>

{#if warning !== undefined}
<p>Warning: {warning}</p>
<p><button type="button" on:click={ () => reset_warning() }>reset warning</button></p>
{:else}
<!-- LOGIN SECTION -->
<section class="login-section">
    <button type="button" class="login-btn" on:click={ (ev) => get_wallet_addr(ev) }>🔓 Log in with Web3</button>
    <span class="instruction">
      Ensure to have an Ethereum based wallet installed i.e MetaMask. Change the network and account in the wallet and 
      click the button again to update the app's state.
    </span>
</section>

<!-- DASHBOARD SECTION -->
<section class="dashboard-section">
    <h3 class="wallet-status">Wallet available</h3>
    {#if walletnetwork !== undefined && walletaddr != undefined}
    <h3 class="wallet-address-heading">
      Wallet address:
      <span class="typewriter">{networkname}({walletnetwork})</span> / 
      <span class="typewriter">{walletaddr}</span>
    </h3>
    {#if walletbalance !== undefined}
    <h3 class="wallet-balance-heading">
     Balance:
      <span class="wallet-balance">{walletbalance}</span>
    </h3>
    {/if}
    <p><button type="button" on:click={ (ev) => get_wallet_balance(ev) }>refresh balance</button></p>
    {/if}
    <p><button type="button" class="logout-btn" on:click={ () => wallet_logout() }>🔐 Log out</button></p>
</section>
{/if}

<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
