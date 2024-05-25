<script lang="ts">
  import { fade } from 'svelte/transition';
  import { createEventDispatcher, onDestroy, onMount } from 'svelte';

  export let zIndex = 1;

  export let viewport: Element;

  export let contents: Element;

  export let hideAfter: number = 1000;

  export let alwaysVisible: boolean = false;

  export let initiallyVisible: boolean = false;

  export let margin: { top?: number; right?: number; bottom?: number; left?: number; } = {};

  export let vTrackIn: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 100 });

  export let vTrackOut: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 300 });

  export let vThumbIn: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 100 });

  export let vThumbOut: (node: Element) => import('svelte/transition').TransitionConfig = (node): import('svelte/transition').TransitionConfig => fade(node, { duration: 300 });

  const dispatch = createEventDispatcher();

  let vTrack: Element;
  let vThumb: Element;

  let startTop = 0;
  let startY = 0;
  let timer = 0;
  let windowScrollEnabled = false;
  let interacted = false;

  $: teardownViewport = setupViewport(viewport);
  $: teardownContents = setupContents(contents);
  $: teardownTrack = setupTrack(vTrack);
  $: teardownThumb = setupThumb(vThumb);

  $: marginTop = margin.top ?? 0;
  $: marginBottom = margin.bottom ?? 0;
  $: marginRight = margin.right ?? 0;
  $: marginLeft = margin.left ?? 0;

  $: wholeHeight = viewport?.scrollHeight ?? 0;
  $: scrollTop = viewport?.scrollTop ?? 0;
  $: trackHeight = viewport?.clientHeight ?? 0 - (marginTop + marginBottom);
  $: thumbHeight = wholeHeight > 0 ? (trackHeight / wholeHeight) * trackHeight : 0;
  $: thumbTop = wholeHeight > 0 ? (scrollTop / wholeHeight) * trackHeight : 0;

  $: scrollable = wholeHeight > trackHeight;
  $: visible = scrollable && (alwaysVisible || initiallyVisible);

  function setupViewport(viewport: Element) {
    if (!viewport) return;

    teardownViewport?.();

    if (typeof window.ResizeObserver === 'undefined') {
      throw new Error('window.ResizeObserver is missing.');
    }

    windowScrollEnabled = document.scrollingElement === viewport;

    // `document.scrollingElement` has the addEventListener function but scroll events wont occur.
    // so we should register the scroll listener to document.
    const element = windowScrollEnabled ? document : viewport;

    element.addEventListener('scroll', onScroll, { passive: true });

    const observer = new ResizeObserver((entries) => {
      for (const _entry of entries) {
        wholeHeight = viewport?.scrollHeight ?? 0;
        trackHeight = viewport?.clientHeight - (marginTop + marginBottom) ?? 0;
      }
    })

    observer.observe(viewport);

    return () => {
      element.removeEventListener('scroll', onScroll);
      observer.unobserve(contents);
      observer.disconnect();
    }
  }

  function setupTrack(track: Element) {
    if (!track) return;

    teardownTrack?.();

    vTrack.addEventListener('mouseenter', onTrackEnter);
    vTrack.addEventListener('mouseleave', onTrackLeave);
    return () => {
      vTrack.removeEventListener('mouseenter', onTrackEnter);
      vTrack.removeEventListener('mouseleave', onTrackLeave);
    }
  }

  function setupThumb(thumb: Element) {
    if (!thumb) return;

    teardownThumb?.();

    vThumb.addEventListener('mousedown', onThumbDown, { passive: true });
    vThumb.addEventListener('touchstart', onThumbDown, { passive: true });

    return () => {
      vThumb.removeEventListener('mousedown', onThumbDown);
      vThumb.removeEventListener('touchstart', onThumbDown);
    }
  }

  function setupContents(contents: Element) {
    if (!contents) return;

    teardownContents?.();

    if (typeof window.ResizeObserver === 'undefined') {
      throw new Error('window.ResizeObserver is missing.');
    }

    const observer = new ResizeObserver((entries) => {
      for (const _entry of entries) {
        wholeHeight = viewport?.scrollHeight ?? 0;
      }
    })

    observer.observe(contents);

    return () => {
      observer.unobserve(contents);
      observer.disconnect();
    }
  }

  function setupTimer() {
    timer = window.setTimeout(() => {
      visible = (scrollable && (alwaysVisible || (initiallyVisible && !interacted))) || false;
      dispatch('hide');
    }, hideAfter)
  }

  function clearTimer() {
    if (timer) {
      window.clearTimeout(timer);
      timer = 0;
    }
  }

  function onScroll() {
    if (!scrollable) return;

    clearTimer();
    setupTimer();

    visible = alwaysVisible || (initiallyVisible && !interacted) || true;
    scrollTop = viewport?.scrollTop ?? 0;

    interacted = true;

    dispatch('show');
  }

  function onTrackEnter() {
    clearTimer();
  }

  function onTrackLeave() {
    clearTimer();
    setupTimer();
  }

  function onThumbDown(event: MouseEvent | TouchEvent) {

    startTop = viewport.scrollTop;
    //startY = event.changedTouches ? event.changedTouches[0].clientY : event.clientY

    if (event instanceof MouseEvent) {
      startY = event.clientY;
    } else if (event instanceof TouchEvent) {
      startY = event.changedTouches[0].clientY;
    }

    document.addEventListener('mousemove', onThumbMove);
    document.addEventListener('touchmove', onThumbMove);
    document.addEventListener('mouseup', onThumbUp);
    document.addEventListener('touchend', onThumbUp);
  }

  function onThumbMove(event: MouseEvent | TouchEvent) {

    //const clientY = event.changedTouches ? event.changedTouches[0].clientY : event.clientY
    let clientY = 0;
    if (event instanceof MouseEvent) {
      clientY = event.clientY;
    } else if (event instanceof TouchEvent) {
      clientY = event.changedTouches[0].clientY;
    }
    const ratio = wholeHeight / trackHeight;

    viewport.scrollTop = startTop + ratio * (clientY - startY);
  }

  function onThumbUp(event: MouseEvent | TouchEvent) {

    startTop = 0;
    startY = 0;

    document.removeEventListener('mousemove', onThumbMove);
    document.removeEventListener('touchmove', onThumbMove);
    document.removeEventListener('mouseup', onThumbUp);
    document.removeEventListener('touchend', onThumbUp);
  }

  onMount(() => {
    viewport = viewport ?? document.scrollingElement;
    contents = contents ?? document.body;
  })

  onDestroy(() => {
    teardownViewport?.();
    teardownContents?.();
    teardownTrack?.();
    teardownThumb?.();
  })
</script>

{#if visible}
  <div
    class="sv-scrollbar"
    class:fixed={windowScrollEnabled}
    style="height: {trackHeight}px; margin: {marginTop}px {marginRight}px {marginBottom}px {marginLeft}px; z-index: {zIndex};">
    <div
      bind:this={vTrack}
      class="sv-track"
      style="height: {trackHeight}px"
      in:vTrackIn
      out:vTrackOut />
    <div
      bind:this={vThumb}
      class="sv-thumb"
      style="height: {thumbHeight}px; top: {thumbTop}px"
      in:vThumbIn
      out:vThumbOut />
  </div>
{/if}

<style>
  .sv-scrollbar {
    position: absolute;
    top: 0;
    right: 0;
    width: var(--svrollbar-track-width, 10px);
  }

  .sv-scrollbar.fixed {
    position: fixed;
  }

  .sv-track {
    position: absolute;
    top: 0;
    right: 0;
    border-radius: var(--svrollbar-track-radius, initial);
    width: var(--svrollbar-track-width, 10px);
    opacity: var(--svrollbar-track-opacity, 1);
    background: var(--svrollbar-track-background, initial);
    box-shadow: var(--svrollbar-track-shadow, initial);
  }

  .sv-thumb {
    position: relative;
    margin: 0 auto;
    border-radius: var(--svrollbar-thumb-radius, 0.25rem);
    width: var(--svrollbar-thumb-width, 6px);
    opacity: var(--svrollbar-thumb-opacity, 0.5);
    background: var(--svrollbar-thumb-background, gray);
    box-shadow: var(--svrollbar-thumb-shadow, initial);
  }
</style>
