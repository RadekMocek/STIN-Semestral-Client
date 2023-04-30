<script>
    import Home from "./lib/Home.svelte";
    import Router from "svelte-spa-router";

    const routes = {
        "/": Home
    };
</script>

<main>
    <Router {routes} />
</main>

