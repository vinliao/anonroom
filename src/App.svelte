<script>
	export let name;

	let message;

	// get data from relay over here
	let texts = ["This is first text", 
		"This is second text",
		"This is third text this is third text this is third text this is third text",
		"This is fourth text",
	];

	function submit(event) {
		texts.push(message);
		// force re-render
		texts = texts;
		message = "";
	}
</script>

<main>
	<div class="d-flex justify-content-center">
		<div class="d-flex flex-column align-items-stretch flex-shrink-0 bg-white" style="width: 576px; height: 100vh;">
			<div class="d-flex justify-content-between mb-3">
			<a href="/" class="d-flex p-3 link-dark text-decoration-none">
				<span class="fw-bold">anonroom</span>
			</a>

			<!-- todo: point to github -->
			<a href="https://github.com" class="d-flex p-3 link-dark text-decoration-none">
				<span class="fw-light">about</span>
			</a>
			</div>


			<div class="list-group list-group-flush border-bottom overflow-auto">
				{#each texts as text}
					<div class="list-group-item py-3 lh-tight" aria-current="true">
						<div class="d-flex w-100 align-items-center justify-content-between">
							<span class="fw-bold">anon says:</span>
							<span class="fw-light">Wed</span>
						</div>
						<div class="col-10 mb-1 small">{text}</div>
					</div>
				{/each}
			</div>

			<form on:submit|preventDefault="{submit}">
				<div class="input-group mb-3"> 
					<input type="text" class="form-control" bind:value="{message}" placeholder="What's on your mind, anon?"> 
					<button class="btn btn-outline-secondary" type="submit">Send</button>
				</div>
			
			</form>
		</div>

	</div>
</main>