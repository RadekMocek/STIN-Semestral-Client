<script>
    import axios from "axios";
    import { push } from "svelte-spa-router";
    import LoadingWheel from "./LoadingWheel.svelte";

    const SERVER_URL = import.meta.env.VITE_SERVER_URL;

    let username = "";
    let password = "";

    let loading = false;
    let errorMessage = "";

    const loginSubmit = () => {
        loading = true;
        errorMessage = "" 

        let credentials = btoa(`${username}:${password}`);

        axios
            .post(SERVER_URL + "/login", null, {
                headers: {
                    Authorization: `Basic ${credentials}`,
                },
            })
            .then((response) => {
                console.log(response.data);                             
            })
            .catch((error) => {
                console.error(error.response.data);
                errorMessage = error.response.data.message;
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
        <input bind:value={username} type="text" id="username" name="username" />
        <br />
        <label for="password">Heslo:</label>
        <br />
        <input bind:value={password} type="password" id="password" name="password" />
        <br /><br />
        <button type="submit" disabled={!username || !password || loading}>Přihlásit se</button>
        {#if loading}
            <LoadingWheel />
        {/if}
    </fieldset>
</form>

<style>
    h2 {
        text-align: center;
    }

    fieldset {
        min-width: 350px;
        width: 25vw;
        padding: 1rem;
    }

    .error {
        background-color: brown;
        padding: 0.75rem 1.5rem;
        color: white;
        margin-bottom: 1rem;
    }
</style>
