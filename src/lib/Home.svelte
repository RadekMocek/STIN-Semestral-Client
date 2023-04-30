<script>
    import moneyMouthFace from "/money-mouth-face.png";
    import bank from "/bank.png";

    import { onMount } from "svelte";

    let isTokenInStorage = false;    

    onMount(() => {
        let token = localStorage.getItem("jwt");
        if (token !== null) {
            isTokenInStorage = true;
        }
    });
</script>

{#if isTokenInStorage}
    <p>Přihlášen</p>
{:else}
    <div>
        <img src={bank} class="logo" alt="Bank Logo" />
        <img src={moneyMouthFace} class="logo" alt="Bank Logo" />
    </div>

    <h1>STIN Bank</h1>

    <div class="card">
        <button>Přihlásit se</button>
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
        filter: drop-shadow(0 0 2em #646cffaa);
    }
</style>
