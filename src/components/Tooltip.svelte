<script>
    export let text = "";
    export let visible = false;
    export let x = 0;
    export let y = 0;
    export let left;
</script>

{#if visible}
    <div
        class="tooltip"
        style="
      top:{y}px;
      left:{x}px;
      transform:{left ? 'translateX(-100%)' : 'translateX(0)'}"
    >
        {text}
    </div>
{/if}

<style>
    .tooltip {
        position: absolute;
        background: white;
        padding: 0.5rem 0.5rem;
        border-radius: 6px;
        pointer-events: none;
        max-width: 180px;
        box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
        font-size: 0.78rem;
        line-height: 1.1;
        font-family:
            system-ui,
            -apple-system,
            BlinkMacSystemFont,
            "Segoe UI",
            Roboto,
            Helvetica,
            Arial,
            sans-serif;
    }
</style>
