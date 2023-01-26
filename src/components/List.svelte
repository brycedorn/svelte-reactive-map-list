<style>
  #list-items {
    display: flex;
    flex-wrap: wrap;
    overflow: scroll;
    scroll-snap-type: y proximity;
  }

  .list-item {
    font-size: 1.2em;
    line-height: 1.5em;
    padding: 40px;
    scroll-snap-align: start;
    width: 100%;
  }

  a {
    color: #000;
    text-decoration-color: #ccc;
  }

  a:hover {
    text-decoration-color: #000;
  }

  .list-item p {
    margin-bottom: 1.4em;
  }

  #list-item-0 {
    scroll-snap-align: none;
  }

  :global(.list-item p:last-child) {
    margin: 0;
  }

  .separator {
    border-bottom: #ccc solid 1px;
    width: 100%;
    margin: 0 40px;
  }

  .head {
    margin: 0 40px;
    scroll-snap-align: none;
  }

  h1 {
    font-size: 3.4em;
  }

  h2 {
    font-size: 2em;
    margin-top: 0.4em;
  }

  h1, h2 {
    font-family: 'Manrope', serif;
  }

  p {
    font-family: Georgia, 'Times New Roman', Times, serif;
  }

  img {
    margin-left: 0.15em;
  }
</style>

<script>
  import { onMount, onDestroy } from 'svelte';
  import inView from 'in-view';
  import { listItems, sprudgeArticle } from './consts.js';
  import { activeListItem, activeMapItem } from './stores.js';
  import InstagramEmbed from './InstagramEmbed.svelte';
  import flag from '../img/amsterdam.svg';

  // Define the ref
  let listRef;

  onMount(async () => {
    // Set a nicer offset so it's not a hard cutoff
    inView.offset(200);

    listRef.addEventListener('scroll', function() {
      // Active list item is top-most fully-visible item
      const visibleListItems = Array.from(
        document.getElementsByClassName('list-item')
      ).map(inView.is);
      // If it's a new one, update active list item
      const topMostVisible = visibleListItems.indexOf(true);
      if (topMostVisible !== $activeMapItem) {
        activeMapItem.set(topMostVisible);
      }
    });
  });

  // Update list scroll position when active list item is updated via map
  const unsubscribeActiveListItem = activeListItem.subscribe(
    newActiveListItem => {
      if (listRef) {
        listRef.scrollTop = document.getElementById(
          `list-item-${newActiveListItem}`
        ).offsetTop;
      }
    }
  );

  // Remove listener on unmount
  onDestroy(unsubscribeActiveListItem);
</script>

<div id="list-items" bind:this="{listRef}">
  <div class="head">
    <h1>
      Where To Drink Coffee In Amsterdam
      <img width="64" src={flag} alt="Flag of Amsterdam" />
    </h1>
    <p>
      Created as part of a <a href="https://svelte.dev/">Svelte</a>
      tutorial on <a href="https://dev.to/bryce/an-interactive-scrolling-map-list-in-svelte-34c3">dev.to</a>.
      View source on <a href="https://github.com/brycedorn/svelte-reactive-map-list">GitHub</a>.
    </p>
  </div>
  <div class="separator" />
  {#each listItems as listItem, index}
    <div class="list-item" id="list-item-{index}">
      <a href="{listItem.website}">
        <h2>{listItem.name}</h2>
      </a>
      <p>{listItem.description}</p>
      <InstagramEmbed id={listItem.instagramPostId} name={listItem.name} />
    </div>
    {#if index < listItems.length}
      <div class="separator" />
    {/if}
  {/each}
</div>
