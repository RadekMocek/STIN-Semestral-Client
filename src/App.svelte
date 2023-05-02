<script>
    import Home from "./lib/Home.svelte";
    import Login from "./lib/Login.svelte";
    import PaymentHistory from "./lib/PaymentHistory.svelte";
    import Router from "svelte-spa-router";

    const routes = {
        "/": Home,
        "/login": Login,
        "/payment/history/:iban": PaymentHistory,
    };
</script>

<main>
    <Router {routes} />
</main>
