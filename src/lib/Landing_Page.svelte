<script>
    import Globe from './Globe.svelte';
    import InfoHover from './InfoHover.svelte';

    let selectedPlace = $state(null);

    function handlePlaceClick(place) {
        selectedPlace = place;
    }

    function handleBack() {
        selectedPlace = null;
    }

    let time = $state();
    function updateItalyTime() {
      time = new Date().toLocaleTimeString("en-GB", {
        timeZone: "Europe/Rome",
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit",
      });
    }

    updateItalyTime();               // show immediately
    setInterval(updateItalyTime, 1000); // refresh every second
</script>

<div class="landing-page">
    <div class="landscape">
        <img src="/italian-landscape.png" alt="Italian Landscape" />
        <div class="landscape-overlay"></div>
    </div>

    <div class="globe-pos">
        <div class="globe-wrapper">
            <Globe onplaceclick={handlePlaceClick} />
        </div>
    </div>

    {#if selectedPlace}
        <InfoHover place={selectedPlace} onback={handleBack} />
    {/if}

    <div class="left-top">
        <div class="title-row">
            <h1 class="title">The Italian Atelier</h1>
            <img class="flag" src="/flag.png" alt="Italy Flag" />
        </div>
        <p class="quote">“Crafted journeys through Italy, stitched together one unforgettable moment at a time.”</p>
        <p class="instruction">Click on a place on the globe to explore more about it</p>
    </div>

    <div class="right-top">
        <span class="time">{time}</span>
        <span class="time-label">Italian Time</span>
    </div>
</div>

<style>
    * {
        box-sizing: border-box;
        font-family: "EB Garamond", serif;
    }

    .landing-page {
        position: relative;
        width: 100vw;
        height: 100vh;
        background-color: #ffffff;
        overflow: hidden;
        font-family: sans-serif;
    }

    .landscape {
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
        height: 50%;
        pointer-events: none;
        z-index: 1;
    }

    .landscape img {
        width: 100%;
        height: 100%;
        object-fit: cover;
        display: block;
    }

    .landscape-overlay {
        position: absolute;
        inset: 0;
        background: linear-gradient(180deg, rgba(255, 255, 255, 1) 0%, rgba(255, 255, 255, 0) 100%);
    }

    .globe-pos {
        position: absolute;
        width: 100%;
        height: 100%;
        z-index: 3;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .globe-wrapper {
        inset: 0;
        width: 80%;
        height: 80%;
        z-index: 2;
    }

    .left-top {
        position: absolute;
        top: 24px;
        left: 24px;
        width: 234px;
        display: flex;
        flex-direction: column;
        gap: 8px;
        z-index: 3;
    }

    .title-row {
        display: flex;
        align-items: center;
        gap: 8px;
    }

    .title {
        font-size: 24px;
        font-weight: 400;
        color: #000000;
        margin: 0;
        line-height: 1.2;
    }

    .flag {
        width: 30px;
        height: 20px;
        object-fit: cover;
        display: block;
    }

    .quote {
        font-size: 14px;
        font-weight: 400;
        color: #000000;
        opacity: 0.8;
        line-height: 1.4;
        margin: 0;
    }

    .instruction {
        font-size: 14px;
        font-weight: 400;
        color: #000000;
        opacity: 0.8;
        line-height: 1.4;
        margin: 0;
    }

    .right-top {
        position: absolute;
        top: 24px;
        right: 24px;
        display: flex;
        flex-direction: column;
        align-items: flex-end;
        z-index: 3;
    }

    .time {
        font-size: 24px;
        font-weight: 400;
        color: #000000;
        text-align: right;
        line-height: 1.2;
    }

    .time-label {
        font-size: 14px;
        font-weight: 400;
        color: #000000;
        opacity: 0.8;
        text-align: right;
        line-height: 1.4;
    }
</style>
