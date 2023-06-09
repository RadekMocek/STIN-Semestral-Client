<script>
    // @ts-nocheck
    import { onMount } from "svelte";
    import { push } from "svelte-spa-router";
    import axios from "axios";
    import Navbar from "./Navbar.svelte";
    import LoadingWheel from "./LoadingWheel.svelte";

    const SERVER_URL = import.meta.env.VITE_SERVER_URL;

    let selectedCurrency = "CZK";
    let amount = 100;

    $: converted = round6(amount * exchangeRates[selectedCurrency]);
    $: newBalance = round6(selectedAccount.balance + (hasAccountInSelectedCurrency ? amount : converted));
    $: overdraftAvailable =
        amount < 0 &&
        -(hasAccountInSelectedCurrency ? amount : converted) < round6(selectedAccount.balance + selectedAccount.balance * 0.1);
    $: newCzkBalance = round6(czkAccount.balance + converted);
    $: outgoingWithConversion = newBalance < 0 && amount < 0 && selectedCurrency !== "CZK" && hasAccountInSelectedCurrency;
    $: outgoingWithConversionValid = outgoingWithConversion && newCzkBalance > -round6(czkAccount.balance * 0.1);
    $: czkOverdraft = outgoingWithConversionValid && newCzkBalance < 0;

    let exchangeRates = [];
    let userAccounts = [];
    let selectedAccount = {};
    let czkAccount = {};
    let hasAccountInSelectedCurrency = true;

    let loading = false;
    let paymentLoading = false;
    let errorMessage = "";
    let paymentEnd = false;
    let paymentErrorMessage = "";
    let paymentSuccessMessage = "";

    onMount(() => {
        GetExchangeRates();
        GetUserAccounts();
    });

    const Logout = () => {
        push("/");
    };

    const round6 = (num) => {
        return +(Math.round(num + "e+6") + "e-6");
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
                userAccounts = response.data;
                czkAccount = userAccounts.find((account) => account.currency === selectedCurrency);
                selectedAccount = czkAccount;
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

    const SelectUserAccount = () => {
        let newAccount = userAccounts.find((account) => account.currency === selectedCurrency);
        if (newAccount) {
            selectedAccount = newAccount;
            hasAccountInSelectedCurrency = true;
        } else {
            selectedAccount = czkAccount;
            hasAccountInSelectedCurrency = false;
        }
    };

    const GetExchangeRates = () => {
        loading = true;
        axios
            .get(SERVER_URL + "/exchange_rates")
            .then((response) => {
                exchangeRates = response.data;
            })
            .catch((error) => {
                if (error.response) {
                    errorMessage = error.response.data.message;
                } else {
                    errorMessage = error;
                }
            })
            .then(() => {
                loading = false;
            });
    };

    const PaySubmit = () => {
        paymentLoading = true;

        let requestAmount = amount;
        let endpoint = "/payment_incoming";

        if (amount < 0) {
            requestAmount = Math.abs(amount);
            endpoint = "/payment_outgoing";
        }

        axios
            .post(
                SERVER_URL + endpoint,
                { currency: selectedCurrency, amount: requestAmount },
                {
                    headers: {
                        Authorization: `Bearer ${localStorage.getItem("jwt")}`,
                    },
                }
            )
            .then((response) => {
                paymentSuccessMessage = response.data.message;
            })
            .catch((error) => {
                if (error.response) {
                    let responseMessage = error.response.data.message;
                    if (responseMessage === "Neplatný token.") {
                        paymentErrorMessage = "Platnost vašeho přihlášení vypršela. Přihlašte se prosím znovu.";
                    } else {
                        paymentErrorMessage = responseMessage;
                    }
                } else {
                    paymentErrorMessage = error;
                }
            })
            .then(() => {
                paymentLoading = false;
                paymentEnd = true;
            });
    };
</script>

<Navbar />
{#if errorMessage}
    <div class="error">{errorMessage}</div>
{/if}
<div id="content">
    <h3>
        Nová platba {#if amount > 0}na účet{:else}z účtu{/if}:
    </h3>
    {#if loading}
        <LoadingWheel />
    {:else}
        <div class="accountBoxUnclikable">
            <h4>{outgoingWithConversion && !overdraftAvailable ? czkAccount.iban : selectedAccount.iban}</h4>
            <h3>
                {outgoingWithConversion && !overdraftAvailable ? czkAccount.balance : selectedAccount.balance}
                {outgoingWithConversion && !overdraftAvailable ? "CZK" : selectedAccount.currency}
            </h3>
        </div>
        <form on:submit|preventDefault={PaySubmit}>
            <fieldset>
                <legend>Platba</legend>
                <input bind:value={amount} disabled={paymentLoading || paymentEnd} type="number" step="0.01" />
                <select bind:value={selectedCurrency} on:change={SelectUserAccount} disabled={paymentLoading || paymentEnd}>
                    {#each Object.keys(exchangeRates).filter((key) => key !== "_Date") as currency}
                        <option value={currency}>{currency}</option>
                    {/each}
                </select>
                <br />
                <p>
                    {#if amount > 0}Jedná se o <b>příchozí</b> platbu.
                    {:else if amount < 0}Jedná se o <b>odchozí</b> platbu.
                    {:else}Kladná částa znamená příchozí a záporná odchozí platbu.
                    {/if}
                </p>
                {#if amount !== 0 && amount !== null}
                    {#if !hasAccountInSelectedCurrency}
                        <p>
                            Účet v této měně nemáte, pro převod na CZK budou použity kurzy z data <b>{exchangeRates._Date}</b>:
                            <br />
                            <code>{amount} {selectedCurrency} = {converted} CZK</code>
                        </p>
                    {/if}
                    <p>Na účtu budete mít {newBalance} {selectedAccount.currency}.</p>
                    {#if newBalance < 0 && amount < 0}
                        {#if overdraftAvailable}
                            <p>
                                Následně bude odečten jednorázový poplatek za kontokorent {round6(newBalance * 0.1)}
                                {selectedAccount.currency}.
                            </p>
                        {:else}
                            <p class="error">Nedostatek prostředků pro provedení platby.</p>
                            {#if selectedCurrency !== "CZK" && hasAccountInSelectedCurrency}
                                Platba bude provedena z CZK účtu budou použity kurzy z data <b>{exchangeRates._Date}</b>:
                                <br />
                                <code>{amount} {selectedCurrency} = {converted} CZK</code>
                                <p>Na účtu budete mít {newCzkBalance} CZK.</p>
                                {#if newCzkBalance < 0}
                                    {#if czkOverdraft}
                                        <p>
                                            Následně bude odečten jednorázový poplatek za kontokorent {round6(newCzkBalance * 0.1)} CZK.
                                        </p>
                                    {:else}
                                        <p class="error">Nedostatek prostředků pro provedení platby.</p>
                                    {/if}
                                {/if}
                            {/if}
                        {/if}
                    {/if}
                    {#if outgoingWithConversionValid || (overdraftAvailable && !paymentEnd)}
                        <button type="submit">Provést platbu</button>
                    {:else}
                        <button type="submit" disabled={(newBalance < 0 && amount < 0) || paymentLoading || paymentEnd}
                            >Provést platbu</button
                        >
                    {/if}
                    {#if paymentLoading}
                        <LoadingWheel />
                    {/if}
                    {#if paymentErrorMessage}
                        <p class="error">Platba nebyla provedena:<br />{paymentErrorMessage}</p>
                    {/if}
                    {#if paymentSuccessMessage}
                        <p class="success">
                            {paymentSuccessMessage}
                            {#if outgoingWithConversionValid}
                                <a href={null} on:click={() => push(`/payment/history/${czkAccount.iban}`)}>Přehled účtu.</a>
                            {:else}
                                <a href={null} on:click={() => push(`/payment/history/${selectedAccount.iban}`)}>Přehled účtu.</a>
                            {/if}
                        </p>
                    {/if}
                {/if}
            </fieldset>
        </form>
    {/if}
</div>

<style>
    fieldset {
        min-width: 350px;
        width: 666px;
        padding: 1rem;
        margin: auto;
        text-align: left;
    }

    input {
        width: 80%;
    }

    select {
        width: 16%;
    }

    .success {
        background-color: green;
        padding: 0.75rem 1.5rem;
        color: white;
        margin-bottom: 1rem;
    }
    .success a {
        color: wheat;
        cursor: pointer;
        text-decoration: underline;
    }
    .success a:hover {
        color: antiquewhite;
    }

    @media only screen and (max-width: 704px) {
        fieldset {
            max-width: fit-content;
        }
    }
</style>
