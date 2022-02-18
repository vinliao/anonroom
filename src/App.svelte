<script>
	export let name;
</script>

<main class="container">
	<div class="nav">
		<h1>anonroom</h1>
		<p>about</p>
	</div>

	<div class="content">
		<p>content1</p>
		<p>conten2</p>
		<p>content3</p>
	</div>

	<div class="input-box">
		<input type="text">
		<button>send</button>
	</div>
</main>

<style>
	.container {
	  max-width: 55ch;
		margin: 0 auto;
		padding: 1rem;	
	}

	.nav {
		display: flex;
		margin: 0 auto;
		padding: 1rem;
	}

	.content {
		height: 100vh;
		display: flex;
		flex-direction: column-reverse;
		justify-content: flex-start;
	}

</style>