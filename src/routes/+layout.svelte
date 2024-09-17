<script>
  import { ModeWatcher } from "mode-watcher";

  import "../app.css";
</script>

<ModeWatcher />

<svelte:head>
  <title>NDOSS</title>
</svelte:head>

<div class="min-h-screen md:w-2/3 lg:w-1/2 mx-auto">
  <slot></slot>
</div>
