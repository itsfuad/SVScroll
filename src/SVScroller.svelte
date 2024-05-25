<script lang="ts">
  import { fade } from 'svelte/transition'
  import Svrollbar from './SVScrollbar.svelte'

  export let width = '10rem';
  export let height = '10rem';
  export let zIndex = 1;

  export let hideAfter = 1000;

  export let alwaysVisible: boolean = false;

  export let initiallyVisible: boolean = false;

  export let margin: { top?: number; right?: number; buttom?: number; left?: number; } = {}

  export let vTrackIn: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 100 })

  export let vTrackOut: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 300 })

  export let vThumbIn: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 100 })

  export let vThumbOut: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 300 })

  let viewport: Element;
  let contents: Element;
</script>

<div class="svlr-wrapper" style="width: {width}; height: {height}">
  <div bind:this={viewport} class="svlr-viewport" style="width: {width}; height: {height}">
    <div bind:this={contents} class="svlr-contents">
      <slot />
    </div>
  </div>
  <Svrollbar
    {viewport}
    {contents}
    {hideAfter}
    {alwaysVisible}
    {initiallyVisible}
    {margin}
    {vTrackIn}
    {vTrackOut}
    {vThumbIn}
    {vThumbOut}
    {zIndex}
    on:show
    on:hide />
</div>

<style>
  .svlr-wrapper {
    position: relative;
  }

  .svlr-viewport {
    position: relative;
    overflow: scroll;
    box-sizing: border-box;

    /* hide scrollbar */
    -ms-overflow-style: none;
    scrollbar-width: none;
  }

  .svlr-viewport::-webkit-scrollbar {
    /* hide scrollbar */
    display: none;
  }
</style>
