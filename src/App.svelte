<script>
  import IFCDate from "./IFCDate.svelte";

  let selected = new Date();
</script>

<p>{selected}</p>

<IFCDate bind:date={selected} />
