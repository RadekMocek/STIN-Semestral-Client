<script>
    import { breakpoint } from "./MediaQuery.svelte";
    import { push, location as routerLocation } from "svelte-spa-router";

    const Logout = () => {
        localStorage.clear();
        if ($routerLocation === "/") {
            location.reload();
        } else {
            push("/");
        }
    };
</script>

<nav>
    <ul>
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <li>
            <h2 on:click={() => push("/")}>
                {#if $breakpoint === "small"}STIN{:else}STIN Bank{/if}
            </h2>
        </li>
        <li>
            <button on:click={() => push("/")}
                >{#if $breakpoint == "small"}<i class="fa fa-list"></i>{:else}Moje účty{/if}</button
            >
            &nbsp;
            <button on:click={() => push("/payment/new")}
                >{#if $breakpoint == "small"}<i class="fa fa-credit-card"></i>{:else}Zadat platbu{/if}</button
            >
            &nbsp;
            <button on:click={Logout}
                >{#if $breakpoint == "small"}<i class="fa fa-sign-out"></i>{:else}Odhlásit se{/if}</button
            >
        </li>
    </ul>
</nav>

<style>
    h2 {
        cursor: pointer;
    }

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
