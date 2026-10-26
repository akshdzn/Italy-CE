<script>
    import Globe from './lib/Globe.svelte';
    import LandingPage from './lib/Landing_Page.svelte';
</script>

<main>
    <LandingPage />
    <!-- <Globe /> -->
</main>

<style>
    :global(html, body, #app) {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
        overflow: hidden;
    }
    main {
        width: 100vw;
        height: 100vh;
    }
</style>
