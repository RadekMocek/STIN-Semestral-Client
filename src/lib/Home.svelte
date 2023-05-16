<script>
    import moneyMouthFace from "/money-mouth-face.png";
    import bank from "/bank.png";
    import LoadingWheel from "./LoadingWheel.svelte";
    import Navbar from "./Navbar.svelte";

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
                if (error.response) {
                    let responseMessage = error.response.data.message;
                    if (responseMessage === "Neplatný token.") {
                        errorMessage = "Platnost vašeho přihlášení vypršela. Přihlašte se prosím znovu.";
                        Logout();
                    }
                } else {
                    errorMessage = error;
                }
            })
            .then(() => {
                loading = false;
            });
    };
</script>

{#if isTokenInStorage}
    <Navbar />
    {#if errorMessage}
        <div class="error">{errorMessage}</div>
    {/if}
    <div id="content">
        <h3>Vaše účty:</h3>
        {#if loading}
            <LoadingWheel />
        {:else}
            {#each userAccouts as { balance, currency, iban }}
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <div class="accountBox" on:click={() => push(`/payment/history/${iban}`)}>
                    <h4>{iban}</h4>
                    <h3>
                        {balance}
                        {currency}<!--<button on:click|stopPropagation={() => push(`/payment/new/${iban}`)}>💳</button>-->
                    </h3>
                </div>
            {/each}
            <p><i>Kliknutím na jeden z účtů zobrazíte jeho historii plateb.</i></p>
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
    .logo {
        height: 6em;
        padding: 1.5em;
        will-change: filter;
        transition: filter 300ms;
    }
    .logo:hover {
        filter: drop-shadow(0 0 2em var(--blue));
    }

    .accountBox {
        background-color: var(--grayDark);
        padding: 0.66rem 2rem;
        margin: 1rem auto;
        max-width: 640px;
        display: flex;
        justify-content: space-between;
        cursor: pointer;
    }
    .accountBox:hover {
        filter: drop-shadow(0 0 1em var(--grayDark));
    }
    /*
    .accountBox button {
        padding: 0;
        margin: 0 0 0 1rem;
    }
    */
</style>
