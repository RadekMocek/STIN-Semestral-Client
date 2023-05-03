<script>
    import { breakpoint } from "./MediaQuery.svelte";
    import { onMount } from "svelte";
    import { push } from "svelte-spa-router";
    import axios from "axios";
    import Navbar from "./Navbar.svelte";
    import LoadingWheel from "./LoadingWheel.svelte";

    const SERVER_URL = import.meta.env.VITE_SERVER_URL;

    export let params = {};

    let payments = [];
    let userAccount = {};

    let loading = false;
    let errorMessage = "";

    onMount(() => {
        GetUserAccount();
        GetPaymentHistory();
    });

    const Logout = () => {
        push("/");
    };

    const unixTimeToHumanTime = (unixTime) => {
        let date = new Date(unixTime * 1000);
        return date.toLocaleString("cs-CZ", {
            timeZone: "Europe/Prague",
            year: "numeric",
            month: "short",
            day: "numeric",
            hour: "numeric",
            minute: "numeric",
            second: "numeric",
        });
    };

    const GetUserAccount = () => {
        loading = true;
        axios
            .get(SERVER_URL + "/user_bank_accounts", {
                headers: {
                    Authorization: `Bearer ${localStorage.getItem("jwt")}`,
                },
            })
            .then((response) => {
                let userAccounts = response.data;
                userAccount = userAccounts.find((account) => account.iban === params.iban);
            })
            .catch((error) => {
                if (error.response) {
                    let responseMessage = error.response.data.message;
                    if (responseMessage === "Neplatný token.") {
                        Logout();
                    }
                }
                errorMessage = error;
            })
            .then(() => {
                loading = false;
            });
    };

    const GetPaymentHistory = () => {
        loading = true;
        axios
            .get(SERVER_URL + "/payment_history", {
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${localStorage.getItem("jwt")}`,
                },
                params: {
                    iban: `${params.iban}`,
                },
            })
            .then((response) => {
                payments = response.data;
                payments = payments.sort((a, b) => b.timestamp - a.timestamp);
            })
            .catch((error) => {
                if (error.response) {
                    let responseMessage = error.response.data.message;
                    if (responseMessage === "Neplatný token.") {
                        Logout();
                    }
                }
                errorMessage = error;
            })
            .then(() => {
                loading = false;
            });
    };
</script>

<Navbar />
{#if errorMessage}
    <div class="error">{errorMessage}</div>
{/if}
<div id="content">
    <h3>Historie plateb pro účet:</h3>
    {#if loading}
        <LoadingWheel />
    {:else}
        <div class="accountBoxUnclikable">
            <h4>{userAccount.iban}</h4>
            <h3>{userAccount.balance} {userAccount.currency}</h3>
        </div>
        {#each payments as { timestamp, value }}
            <div class="paymentBox">
                <p>
                    <b>{unixTimeToHumanTime(timestamp)}</b>
                    {#if $breakpoint !== "small"}
                        <span>&nbsp;&emsp;{value > 0 ? "Příchozí platba" : "Odchozí platba"}</span>
                    {/if}
                </p>
                <p><b>{value} {userAccount.currency}</b></p>
            </div>
        {/each}
    {/if}
</div>

<style>
    .paymentBox {
        background-color: var(--gray);
        padding: 0 2rem;
        margin: 0.25rem auto;
        max-width: 640px;
        display: flex;
        justify-content: space-between;
    }
</style>
