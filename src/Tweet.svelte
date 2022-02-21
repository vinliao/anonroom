<script>
  export let time;
  export let message;
</script>

<div class="flex justify-between">
  <span class="font-bold">anon says:</span>
  <span class="font-light">{time}</span>
</div>
<div class="mb-2">{message}</div>
<div class="flex mb-10">
  <a href="/">
    <span class="font-mono underline">reply</span>
  </a>
  <div class="flex-1" />
</div>
