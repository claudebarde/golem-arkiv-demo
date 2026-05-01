<script lang="ts">
  import "../styles/index.scss";
  import favicon from "$lib/assets/favicon.svg";

  let { children } = $props();
</script>

<svelte:head>
  <link rel="icon" href={favicon} />
  <title>Golem Arkiv Demo</title>
</svelte:head>

{@render children()}
