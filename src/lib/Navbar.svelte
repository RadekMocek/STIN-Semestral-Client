<script>
    import { push, location as routerLocation } from "svelte-spa-router";

    const Logout = () => {
        localStorage.clear();
        if ($routerLocation === "/") {
            location.reload();
        }
        else {
            push("/");
        }
    };
</script>

<nav>
    <ul>
        <li><h2>STIN Bank</h2></li>
        <li><button on:click={Logout}>Odhlásit se</button></li>
    </ul>
</nav>

<style>
    nav {
        background-color: var(--grayDark);
        width: 100vw;
        padding: 0;
        margin: 0;
    }
    nav ul {
        margin: 0;
        padding: 0;
        height: 4rem;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    nav li {
        margin: 0 1.5rem;
        display: inline-block;
        vertical-align: top;
    }
</style>
