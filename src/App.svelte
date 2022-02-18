<script>
	export let name;

	// get data from relay over here
	const texts = ["This is first text", "This is second text", "This is third text what if awzlznlkndsv;landlvkansdl;"];

	function submit() {
		alert('submitting');
	}
</script>

<main>
	<div class="d-flex flex-column align-items-stretch flex-shrink-0 bg-white" style="width: 380px;">
		<a href="/" class="d-flex align-items-center flex-shrink-0 p-3 link-dark text-decoration-none border-bottom">
			<span class="fs-5 fw-semibold">anonroom</span>
		</a>


		<div class="list-group list-group-flush border-bottom scrollarea">
			{#each texts as text}
				<a href="#" class="list-group-item list-group-item-action py-3 lh-tight" aria-current="true">
					<div class="d-flex w-100 align-items-center justify-content-between">
						<strong class="mb-1">anon says:</strong>
						<small>Wed</small>
					</div>
					<div class="col-10 mb-1 small">{text}</div>
				</a>
			{/each}
		</div>

		<form on:submit|preventDefault="{submit}">
			<div class="input-group"> <input type="text" class="form-control" placeholder="What's on your mind, anon?" aria-label="Recipient's username" aria-describedby="button-addon2"> 
				<button class="btn btn-outline-secondary" type="submit" id="button-addon2">Send</button>
			</div>
		
		</form>
	</div>
</main>