<script>
    import Home from "./lib/Home.svelte";
    import Login from "./lib/Login.svelte";
    import Router from "svelte-spa-router";

    const routes = {
        "/": Home,
        "/login": Login,
    };
</script>

<main>
    <Router {routes} />
</main>
