<script lang="ts">
  import { createEventDispatcher } from "svelte";
  import { fade } from "svelte/transition";

  export let duration: number;
  export let scroll: number;
  export let pixelsPerSecond: number;

  let clientWidth: number | undefined;

  let timestampWidth: number;
  $: timestampWidth = clientWidth || 0;

  let timestampCount: number;
  $: timestampCount = Math.floor(timestampWidth / 100);

  let timestampPx: number;
  $: timestampPx = timestampWidth / timestampCount;

  let widthSecs: number;
  $: widthSecs = timestampWidth / pixelsPerSecond;

  let timestampPeriod: number;
  $: timestampPeriod = widthSecs / timestampCount;

  let timestamps: {left: number, mins: string, secs: string}[];
  $: timestamps = new Array(timestampCount)
    .fill(null)
    .map((_, idx) => ({ts: scroll + timestampPeriod * (idx - timestampCount / 2 + 0.5), idx}))
    .filter(({ts}) => ts >= 0 && ts <= duration)
    .map(({ts, idx}) => ({
      left: timestampPx * (idx + 0.5),
      mins: Math.floor(ts / 60).toFixed().padStart(2, "0"),
      secs: (ts % 60).toFixed(3).padStart(6, "0")
    }));

  let hovering = false;
  
  let cursorPosition: number | null = null;
  export let cursorPositionSeconds: number | null;
  $: cursorPositionSeconds = cursorPosition === null ? null : scroll + cursorPosition / pixelsPerSecond;

  let minMousePosition: number;
  $: minMousePosition = -scroll * pixelsPerSecond;

  let maxMousePosition: number;
  $: maxMousePosition = duration * pixelsPerSecond + minMousePosition;

  function mouseEnter() {
    hovering = true;
  }

  function mouseLeave() {
    hovering = false;
    cursorPosition = null;
  }

  function mouseMove(event: MouseEvent) {
    if(hovering) {
      const width = clientWidth === undefined ? event.offsetX : clientWidth;
      const mousePosition = event.offsetX - width / 2;
      cursorPosition = Math.min(Math.max(minMousePosition, mousePosition), maxMousePosition);
    }
  }

  const dispatch = createEventDispatcher();

  function mouseClick(event: MouseEvent) {
    const width = clientWidth === undefined ? event.offsetX : clientWidth;
    const mousePosition = event.offsetX - width / 2;
    const seconds = mousePosition / pixelsPerSecond;
    const scrolledSeconds = scroll + seconds;
    const startAt = Math.min(Math.max(0, scrolledSeconds), duration);
    dispatch("play", startAt);
  }
</script>

<style>
  .timestamps {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 30px;
    cursor: pointer;
  }

  .column {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    pointer-events: none;
  }

  .marker {
    width: 1px;
    height: 100%;
    background-color: gray;
  }

  .cursor {
    position: absolute;
    top: 30px;
    bottom: 0;
    left: 0;
    width: 1px;
    background-color: gray;
    z-index: 2;
  }
</style>

{#if cursorPosition !== null}
<div class="cursor" style={`transform: translateX(calc(${cursorPosition}px - 50% + 50vw))`}/>
{/if}

<div class="timestamps" bind:clientWidth on:mouseover={mouseEnter} on:mouseout|self={mouseLeave} on:mousemove={mouseMove} on:click={mouseClick}>
  {#each timestamps as {left, mins, secs} (left)}
    <div class="column" style={`transform: translateX(calc(${left}px - 50%))`} transition:fade>
      <div class="timestamp">{mins}:{secs}</div>
      <div class="marker"/>
    </div>
  {/each}
</div>