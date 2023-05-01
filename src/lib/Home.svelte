<script>
    import moneyMouthFace from "/money-mouth-face.png";
    import bank from "/bank.png";

    import { push } from "svelte-spa-router";
    import { onMount } from "svelte";

    let isTokenInStorage = false;

    onMount(() => {
        let token = localStorage.getItem("jwt");
        if (token !== null) {
            isTokenInStorage = true;
        }
    });

    const Logout = () => {
        localStorage.clear();
        isTokenInStorage = false;
    };
</script>

{#if isTokenInStorage}
    <nav>
        <ul>
            <li><h2>STIN Bank</h2></li>
            <li><button on:click={Logout}>Odhlásit se</button></li>
        </ul>
    </nav>
{:else}
    <div id="content">
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
        filter: drop-shadow(0 0 2em #646cffaa);
    }
</style>
