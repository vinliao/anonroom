<script>
  export let time;
  export let message;
  export let replied; // a reply object, containing another tweet
</script>

<div class="flex justify-between">
  <span class="font-bold">anon says:</span>
  <span class="font-light text-slate-500">{time}</span>
</div>
{#if replied}
  <!-- <div class="ml-5 p-3 mb-2 bg-gray-50 border-y border-l border-slate-500"> -->
  <div class="my-1 pl-3 border-l-4 border-red-800">
    <div class="flex justify-between">
      <span>anon said:</span>
      <div></div>
    </div>
    <div class="font-light text-slate-600">{replied.message}</div>
  </div>
{/if}
<div class="mb-2">{message}</div>
<!-- there are pr-3 above bc if p-3 is applied to the parent div
the background of replied text will also get padded -->