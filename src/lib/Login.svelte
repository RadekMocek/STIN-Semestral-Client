<script>
    import axios from "axios";
    import { push } from "svelte-spa-router";
    import { onMount } from "svelte";
    import LoadingWheel from "./LoadingWheel.svelte";

    const SERVER_URL = import.meta.env.VITE_SERVER_URL;

    let username = "";
    let password = "";
    let twoFactorCode = "";

    let loading = false;
    let errorMessage = "";
    let successfulLogin = false;

    onMount(() => {
        let token = localStorage.getItem("jwt");
        if (token !== null) {
            push("/");
        }
    });

    const loginSubmit = () => {
        loading = true;
        errorMessage = "";

        let credentials = btoa(`${username}:${password}`);

        axios
            .post(SERVER_URL + "/login", null, {
                headers: {
                    Authorization: `Basic ${credentials}`,
                },
            })
            .then(() => {
                successfulLogin = true;
            })
            .catch((error) => {
                if (error.response) {
                    errorMessage = error.response.data.message;
                }
                else {
                    errorMessage = error;                    
                }
                password = "";
            })
            .then(() => {
                loading = false;
            });
    };

    const twoFactorSubmit = () => {
        loading = true;
        errorMessage = "";

        axios
            .post(SERVER_URL + "/authorize", { username: username, code: twoFactorCode })
            .then((response) => {
                localStorage.setItem('jwt', response.data.token);
                push("/")
            })
            .catch((error) => {
                if (error.response) {
                    errorMessage = error.response.data.message;
                }
                else {
                    errorMessage = error;                    
                }
            })
            .then(() => {
                loading = false;
            });
    };
</script>

<h2>Přihlásit se</h2>

{#if errorMessage}
    <div class="error">{errorMessage}</div>
{/if}

<form on:submit|preventDefault={loginSubmit}>
    <fieldset>
        <legend>Přihlášení</legend>
        <label for="username">Uživatelské jméno:</label>
        <br />
        <input bind:value={username} disabled={successfulLogin} type="text" id="username" name="username" />
        <br />
        <label for="password">Heslo:</label>
        <br />
        <input bind:value={password} disabled={successfulLogin} type="password" id="password" name="password" />
        <br /><br />
        <button type="submit" disabled={!username || !password || loading || successfulLogin}>Přihlásit se</button>
        {#if loading && !successfulLogin}
            <LoadingWheel />
        {/if}
    </fieldset>
</form>

{#if successfulLogin}
    <br />
    <p>Na váš e-mail byl odeslán 2fa kód.</p>
    <br />
    <form on:submit|preventDefault={twoFactorSubmit}>
        <fieldset>
            <legend>Dvoufázové oveření</legend>
            <label for="twoFactorCode">Kód z e-mailu:</label>
            <br />
            <input bind:value={twoFactorCode} type="text" id="twoFactorCode" name="twoFactorCode" maxlength="16" />
            <br /><br />
            <button type="submit" disabled={!twoFactorCode}>Potvrdit</button>
            {#if loading}
                <LoadingWheel />
            {/if}
        </fieldset>
    </form>
{/if}

<style>
    h2 {
        text-align: center;
    }

    fieldset {
        min-width: 350px;
        width: 25vw;
        padding: 1rem;
    }    
</style>
