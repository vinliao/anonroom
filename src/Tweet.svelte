<script>
  export let time;
  export let message;
  export let replied; // a reply object, containing another tweet
</script>

<div class="flex justify-between">
  <span class="font-bold">anon says:</span>
  <span class="font-light text-slate-500 pr-3">{time}</span>
</div>
<div class="mb-2">{message}</div>
{#if replied}
  <div class="ml-5 p-3 mb-2 bg-gray-50">
    <div class="flex justify-between">
      <span class="font-bold">anon says:</span>
      <span class="font-light text-slate-500">{replied.time}</span>
    </div>
    <div>{replied.message}</div>
  </div>
{/if}
