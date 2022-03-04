<script>
  export let time;
  export let message;
  export let replied; // a reply object, containing another tweet

  function alertFeature() {
    alert("feature not ready!");
  }
</script>

<div class="flex justify-between">
  <span class="font-bold">anon says:</span>
  <span class="text-slate-500">{time}</span>
</div>
{#if replied}
  <!-- <div class="ml-5 p-3 mb-2 bg-gray-50 border-y border-l border-slate-500"> -->
  <div class="my-1 pl-3 border-l-4 border-red-800">
    <div class="flex justify-between">
      <span>anon said:</span>
      <div></div>
    </div>
    <!-- <div class="text-slate-500 break-words">{replied.message}</div> -->
    {#if replied.message.length < 280}
      <span class="mb-2 break-words">{replied.message}</span>
    {:else}
      <span class="mb-2 break-words">{replied.message.slice(0, 280) + " ... "}</span>
      <button class="font-mono underline" on:click="{alertFeature}">[more]</button>
    {/if}
  </div>
{/if}
{#if message.length < 280}
  <span class="mb-2 break-words">{message}</span>
{:else}
  <span class="mb-2 break-words">{message.slice(0, 280) + " ... "}</span>
  <button class="font-mono underline" on:click="{alertFeature}">[more]</button>
{/if}

<!-- there are pr-3 above bc if p-3 is applied to the parent div
the background of replied text will also get padded -->