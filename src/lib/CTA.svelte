<script>
    let {
        class: className = '',
        style = '',
        onclick = () => {},
        ...restProps
    } = $props();

    function handleClick(e) {
        e.preventDefault();
        // Do nothing when clicked
        if (onclick) onclick(e);
    }
</script>

<button
    type="button"
    class="cta-button {className}"
    style={style}
    onclick={handleClick}
    aria-label="Book a Call"
    data-node-id="46:30"
    data-name="cta"
    {...restProps}
>
    <!-- Badge on bottom left -->
    <div class="cta-badge-wrapper" data-node-id="46:27">
        <div class="cta-badge-rotate">
            <div class="cta-badge-box" data-name="image 3">
                <img src="/cta-badge.png" alt="Welcome to Italy badge" draggable="false" />
            </div>
        </div>
    </div>

    <!-- Center Pill Button -->
    <div class="cta-pill" data-node-id="46:29">
        <span class="cta-text" data-node-id="46:19">Book a Call</span>
    </div>

    <!-- Phone on top right -->
    <div class="cta-phone-wrapper" data-node-id="46:24" data-name="image 2">
        <img src="/cta-phone.png" alt="Telephone handset" draggable="false" />
    </div>
</button>

<style>
    .cta-button {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 201px;
        height: 104px;
        background: transparent;
        border: none;
        padding: 0;
        margin: 0;
        cursor: pointer;
        z-index: 10;
        user-select: none;
        outline: none;
        display: block;
        transition: transform 0.2s cubic-bezier(0.34, 1.56, 0.64, 1);
    }

    .cta-button:hover {
        transform: translate(-50%, -50%) scale(1.04);
    }

    .cta-button:active {
        transform: translate(-50%, -50%) scale(0.97);
    }

    .cta-pill {
        position: absolute;
        left: 26px;
        top: 18px;
        width: 162px;
        height: 51px;
        border-radius: 53px;
        background: linear-gradient(180deg, #ed1b2b 0%, #ff525f 100%);
        box-shadow: 0px 2px 1px rgba(0, 0, 0, 0.25), inset 0px -2px 4px 0px #ed1b2b;
        display: flex;
        align-items: center;
        justify-content: center;
        box-sizing: border-box;
        z-index: 2;
    }

    .cta-text {
        font-family: "EB Garamond", serif;
        font-weight: 400;
        font-size: 20px;
        color: #ffffff;
        white-space: nowrap;
        line-height: normal;
        letter-spacing: 0.01em;
        pointer-events: none;
    }

    .cta-badge-wrapper {
        position: absolute;
        left: 0;
        top: 19px;
        width: 85px;
        height: 85px;
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 3;
        pointer-events: none;
    }

    .cta-badge-rotate {
        transform: rotate(22.34deg);
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .cta-badge-box {
        position: relative;
        width: 65px;
        height: 65px;
    }

    .cta-badge-box img {
        width: 100%;
        height: 100%;
        object-fit: contain;
        display: block;
        pointer-events: none;
    }

    .cta-phone-wrapper {
        position: absolute;
        left: 155px;
        top: 0;
        width: 46px;
        height: 58px;
        z-index: 3;
        pointer-events: none;
    }

    .cta-phone-wrapper img {
        width: 100%;
        height: 100%;
        object-fit: contain;
        display: block;
        pointer-events: none;
    }
</style>
