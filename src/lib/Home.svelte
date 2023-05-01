<script>
    import moneyMouthFace from "/money-mouth-face.png";
    import bank from "/bank.png";
    import LoadingWheel from "./LoadingWheel.svelte";

    import { push } from "svelte-spa-router";
    import { onMount } from "svelte";
    import axios from "axios";

    const SERVER_URL = import.meta.env.VITE_SERVER_URL;

    let userAccouts = [];

    let isTokenInStorage = false;
    let loading = false;
    let errorMessage = "";

    onMount(() => {
        let token = localStorage.getItem("jwt");
        if (token !== null) {
            isTokenInStorage = true;
            GetUserAccounts();
        }
    });

    const Logout = () => {
        localStorage.clear();
        isTokenInStorage = false;
    };

    const GetUserAccounts = () => {
        loading = true;
        axios
            .get(SERVER_URL + "/user_bank_accounts", {
                headers: {
                    Authorization: `Bearer ${localStorage.getItem("jwt")}`,
                },
            })
            .then((response) => {
                userAccouts = response.data;
            })
            .catch((error) => {
                let responseMessage = error.response.data.message;
                if (responseMessage === "Neplatný token.") {
                    errorMessage = "Platnost vašeho přihlášení vypršela. Přihlaště se prosím znovu.";
                    Logout();
                }
            })
            .then(() => {
                loading = false;
            });
    };
</script>

{#if isTokenInStorage}
    <nav>
        <ul>
            <li><h2>STIN Bank</h2></li>
            <li><button on:click={Logout}>Odhlásit se</button></li>
        </ul>
    </nav>
    {#if errorMessage}
        <div class="error">{errorMessage}</div>
    {/if}
    <div id="content">
        <h3>Vaše účty:</h3>
        {#if loading}
            <LoadingWheel />
        {:else}
            {#each userAccouts as { balance, currency, iban }}
                <div class="account">
                    <p>{iban}</p>
                    <p>{balance} {currency}</p>
                </div>
            {/each}
        {/if}
    </div>
{:else}
    <div id="content">
        {#if errorMessage}
            <div class="error">{errorMessage}</div>
        {/if}

        <img src={bank} class="logo" alt="Bank Logo" />
        <img src={moneyMouthFace} class="logo" alt="Bank Logo" />

        <h1>STIN Bank</h1>

        <div class="card">
            <button on:click={() => push("/login")}>Přihlásit se</button>
        </div>
    </div>
{/if}

<style>
    #content {
        text-align: center;
    }

    .logo {
        height: 6em;
        padding: 1.5em;
        will-change: filter;
        transition: filter 300ms;
    }
    .logo:hover {
        filter: drop-shadow(0 0 2em var(--blue));
    }

    .account {
        background-color: var(--grayDark);
        padding: 1rem;
        margin: 1rem auto;
        max-width: 640px;
        display: flex;
        justify-content: space-between;
    }
</style>
