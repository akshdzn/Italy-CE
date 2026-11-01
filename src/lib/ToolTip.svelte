<script>
    let {
        text = 'ROME',
        color = '#18bd67',
        textColor = '#ffffff',
        visible = true,
        class: className = '',
        style = '',
        children,
        ...restProps
    } = $props();
</script>

{#if visible}
    <div
        class="tooltip {className}"
        style="--tooltip-bg: {color}; --tooltip-color: {textColor}; {style}"
        data-node-id="60:5"
        data-name="tooltip"
        {...restProps}
    >
        <div class="tooltip-box" data-node-id="60:3">
            <span class="tooltip-text" data-node-id="60:2">
                {#if children}
                    {@render children()}
                {:else}
                    {text}
                {/if}
            </span>
        </div>
        <svg
            class="tooltip-arrow"
            data-node-id="60:4"
            width="14"
            height="7"
            viewBox="0 0 14 7"
            aria-hidden="true"
        >
            <polygon points="0,0 14,0 7,7" fill="var(--tooltip-bg, #18bd67)" />
        </svg>
    </div>
{/if}

<style>
    .tooltip {
        position: relative;
        display: inline-flex;
        flex-direction: column;
        align-items: center;
        width: fit-content;
        box-sizing: border-box;
        line-height: 0;
        z-index: 10;
        pointer-events: none;
    }

    .tooltip-box {
        background-color: var(--tooltip-bg, #18bd67);
        display: inline-flex;
        align-items: center;
        justify-content: center;
        padding: 3px 11px;
        box-sizing: border-box;
        line-height: normal;
    }

    .tooltip-text {
        font-family: "EB Garamond", serif;
        font-size: 16px;
        font-weight: 400;
        line-height: normal;
        color: var(--tooltip-color, #ffffff);
        text-transform: uppercase;
        white-space: nowrap;
        word-break: break-word;
        user-select: none;
    }

    .tooltip-arrow {
        display: block;
        width: 14px;
        height: 7px;
        margin-top: -1px;
        flex-shrink: 0;
    }
</style>
