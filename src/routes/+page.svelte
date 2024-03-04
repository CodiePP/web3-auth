<script>
    import { onMount } from 'svelte';

    let web3_available = false
    let webauthn_available = false

    onMount ( () => {
        if (window.ethereum) {
            web3_available = true
        }
    })

</script>

<svelte:head>
        <title>Login</title>
        <meta name="description" content="login" />
        <script src="web3.min.js" />
</svelte:head>

<h1>Choose login options</h1>

{#if !!web3_available}
<p>login with your <a href="/web3">web3 wallet</a></p>
{:else}
<p>login with web3 wallet is not available. need to load the browser plugin?</p>
{/if}
