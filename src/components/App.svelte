<style>
  .container {
    height: 100vh;
    display: flex;
  }

  .pane {
    display: flex;
    flex-direction: column;
    max-height: 100vh;
  }

  .left {
    align-items: center;
    width: 60%;
    justify-content: center;
  }

  .right {
    justify-content: space-between;
    width: 40%;
  }
</style>

<script>
  import List from './List.svelte';
  import Map from './Map.svelte';
</script>

<div class="container">
  <div class="pane left">
    <List />
  </div>
  <div class="pane right">
    <Map />
  </div>
</div>
