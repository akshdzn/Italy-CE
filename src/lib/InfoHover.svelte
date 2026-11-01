<script>
    let {
        place = null,
        visible = true,
        name = '',
        image = '',
        matter = [],
        onback = () => {}
    } = $props();

    const placeName = $derived(place?.name ?? name);
    const placeImage = $derived(place?.globeImage ?? image);
    const placeMatter = $derived(place?.matter ?? matter);

    function handleBack() {
        if (onback) onback();
    }
</script>

<svelte:window onkeydown={(e) => { if (e.key === 'Escape') handleBack(); }} />

{#if visible && (place || placeName)}
    <div
        class="info-hover-root"
        data-node-id="61:35"
        data-name="info-hover"
    >
        <button
            type="button"
            class="back-btn"
            data-node-id="61:36"
            onclick={handleBack}
            aria-label="Back to globe"
        >
            ← BACK | {placeName}
        </button>

        <div class="image-container" data-node-id="61:27">
            <div class="image-rotation">
                <div class="image-box" data-name="image">
                    {#if placeImage}
                        <img
                            src={placeImage}
                            alt={placeName}
                            draggable="false"
                            loading="eager"
                            decoding="async"
                            referrerpolicy="no-referrer"
                            crossorigin="anonymous"
                        />
                    {/if}
                </div>
            </div>
        </div>

        <div
            class="info-bullets"
            data-node-id="61:33"
            data-name="info-bullets"
        >
            {#each placeMatter as text, index}
                <p class="bullet-text" data-node-id={`61:${30 + index}`}>{text}</p>
            {/each}
        </div>
    </div>
{/if}

<style>
    .info-hover-root {
        position: absolute;
        top: 50%;
        left: 0;
        width: 100%;
        height: 240px;
        background-color: #ed1b2b;
        transform: translateY(-50%);
        display: flex;
        align-items: center;
        justify-content: flex-end;
        padding-left: clamp(540px, 39vw, 660px);
        padding-right: clamp(40px, 4.5vw, 90px);
        box-sizing: border-box;
        z-index: 100;
        animation: fadeIn 0.22s ease-out;
    }

    @keyframes fadeIn {
        from {
            opacity: 0;
            transform: translateY(-50%) scale(0.99);
        }
        to {
            opacity: 1;
            transform: translateY(-50%) scale(1);
        }
    }

    .back-btn {
        position: absolute;
        top: -40px;
        right: clamp(40px, 4.5vw, 90px);
        background: none;
        border: none;
        padding: 0;
        margin: 0;
        font-family: "EB Garamond", serif;
        font-size: 20px;
        font-weight: 500;
        color: #000000;
        line-height: normal;
        white-space: nowrap;
        cursor: pointer;
        user-select: none;
        transition: opacity 0.15s ease, transform 0.15s ease;
    }

    .back-btn:hover {
        opacity: 0.6;
        transform: translateX(-3px);
    }

    .image-container {
        position: absolute;
        left: clamp(36px, 3.5vw, 72px);
        top: 50%;
        transform: translateY(-50%);
        width: 530px;
        height: 330px;
        display: flex;
        align-items: center;
        justify-content: center;
        pointer-events: none;
    }

    .image-rotation {
        flex: none;
        transform: rotate(-4.68deg);
    }

    .image-box {
        position: relative;
        width: 500px;
        height: 281px;
        overflow: hidden;
        box-shadow: 0 8px 30px rgba(0, 0, 0, 0.22);
    }

    .image-box img {
        position: absolute;
        inset: 0;
        width: 100%;
        height: 100%;
        object-fit: cover;
        display: block;
        pointer-events: none;
    }

    .info-bullets {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: clamp(24px, 2.8vw, 52px);
        width: 100%;
        max-width: 1200px;
        box-sizing: border-box;
    }

    .bullet-text {
        flex: 1 1 0;
        min-width: 200px;
        max-width: 350px;
        margin: 0;
        font-family: "EB Garamond", serif;
        font-size: clamp(18px, 1.25vw, 22px);
        font-weight: 400;
        color: #ffffff;
        line-height: 1.48;
        user-select: text;
        word-break: break-word;
    }

    @media (max-width: 1160px) {
        .info-hover-root {
            height: 260px;
            padding-left: 450px;
            padding-right: 32px;
        }

        .image-container {
            left: 20px;
            width: 420px;
            height: 265px;
        }

        .image-box {
            width: 400px;
            height: 225px;
        }

        .back-btn {
            right: 32px;
            top: -34px;
            font-size: 18px;
        }

        .bullet-text {
            font-size: 17px;
            min-width: 160px;
            line-height: 1.4;
        }
    }

    @media (max-width: 860px) {
        .info-hover-root {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: calc(100% - 32px);
            max-width: 440px;
            height: auto;
            max-height: 86vh;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
            padding: 48px 20px 24px;
            box-shadow: 0 12px 40px rgba(0, 0, 0, 0.4);
            flex-direction: column;
            align-items: center;
            gap: 16px;
        }

        .image-container {
            position: relative;
            left: auto;
            top: auto;
            transform: none;
            width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            pointer-events: none;
            margin: 4px 0 8px;
        }

        .image-rotation {
            flex: 0 1 320px;
            width: 100%;
            max-width: 320px;
            display: flex;
            justify-content: center;
            transform: rotate(-2.5deg);
        }

        .image-box {
            position: relative;
            width: 100%;
            max-width: 320px;
            height: 180px;
            aspect-ratio: 16 / 9;
            overflow: hidden;
            background-color: rgba(255, 255, 255, 0.15);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
        }

        .image-box img {
            position: absolute;
            inset: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
            display: block;
        }

        .info-bullets {
            flex-direction: column;
            gap: 14px;
            width: 100%;
        }

        .bullet-text {
            max-width: 100%;
            min-width: auto;
            font-size: 16px;
            line-height: 1.45;
            text-align: left;
        }

        .back-btn {
            top: 12px;
            right: 16px;
            color: #ffffff;
            font-size: 15px;
            padding: 6px 12px;
        }
    }
</style>
