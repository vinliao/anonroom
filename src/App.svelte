<script>
	import * as secp from "@noble/secp256k1";
	import { format } from 'timeago.js';

	// public and private key of production should be separate from the dev one
	// maybe environment variable?
	// const privateKey = "6ba903b7888191180a0959a6d286b9d0719d33a47395c519ba107470412d2069";
	// const publicKey = "8565b1a5a63ae21689b80eadd46f6493a3ed393494bb19d0854823a757d8f35f";

	// dummy keypair for development
	const privateKey = "b86e8ca23e9594b0d6d78291c23b3fc20e749c464925bdff82fc57ec95e48d0e";
	const publicKey = "6fe50495793e11d3d866b20f45cfe26293a099a25f0533ca031ea16828b10444";
	let inputMessage;

	function toHexString(byteArray) {
		return Array.prototype.map
			.call(byteArray, function (byte) {
				return ("0" + (byte & 0xff).toString(16)).slice(-2);
			})
			.join("");
	}

	async function createEvent() {
		const content = inputMessage;
		const unixTime = Math.floor(Date.now() / 1000);
		const data = [0, publicKey, unixTime, 1, [], content];

		// id is sha256 of data above
		// sig is schnorr sig of id
		const eventString = JSON.stringify(data);
		const eventByteArray = new TextEncoder().encode(eventString);
		const eventIdRaw = await secp.utils.sha256(eventByteArray);
		const eventId = toHexString(eventIdRaw);
		const signatureRaw = await secp.schnorr.sign(eventId, privateKey);
		const signature = toHexString(signatureRaw);

		return {
			id: eventId,
			pubkey: publicKey,
			created_at: unixTime, 
			kind: 1,
			tags: [],
			content: content, 
			sig: signature
		}
	}

	let tweets = [];

	// websocket stuff
	// let url = "ws://localhost:8080";
	let url = "wss://nostr-pub.wellorder.net";
	let socket = new WebSocket(url);

	socket.onopen = function (event) {
		console.log(`Connected to ${url}`)
		socket.send(JSON.stringify(["REQ", "foobar", {"authors": [publicKey]}]));
	};

	// take data and parse here
	socket.onmessage = function (incomingPayload) {
		let payload = JSON.parse(incomingPayload.data);
		let event = payload[2];
		let timestamp = event.created_at + "000"; // add the milliseconds
		let tweet = {"message": event.content, "time": format(timestamp)};
		tweets.unshift(tweet);
		// force re-render
		tweets = tweets;
	}

	async function submit() {
		// submit to relay here
		let event = await createEvent();
		socket.send(JSON.stringify(["EVENT", event]));
		// force re-render
		tweets = tweets;
		inputMessage = null;
	}
</script>

<main class="max-w-lg mx-auto">
	<div class="flex flex-col items-stretch justify-between shrink-0">
			<span class="text-right font-bold text-7xl">anonroom</span>
			<!-- a little trick to make item sit on the right -->
			<div class="flex">
				<div class="flex-1"></div>
				<a href="https://github.com/vinliao/anonroom" class="text-right mb-12">
					<span class="font-light font-mono underline">about</span>
				</a>
			</div>


		<form on:submit|preventDefault="{submit}">
			<div class="flex mb-5"> 
				<input type="text" bind:value="{inputMessage}" rows="3" placeholder="What's on your mind, anon?" class="flex-1">
				<button class="bg-red-800 text-white py-2 px-3 font-mono">send</button>
			</div>
		</form>

		<!-- an example of what the reply ui looks like -->
		<div class="flex justify-between">
			<span class="font-bold">anon says:</span>
			<span class="font-light">2 minutes ago</span>
		</div>
		<div class="mb-2">the reply three two one</div>

		<div class="ml-5 p-3 mb-2 bg-slate-50">
			<div class="flex justify-between">
				<span class="font-bold">anon says:</span>
				<span class="font-light">2 minutes ago</span>
			</div>
			<div>the replied one two three</div>

		</div>

		<div class="flex mb-10">
			<a href="/">
				<span class="font-mono underline">reply</span>
			</a>
			<div class="flex-1"></div>
		</div>


		{#each tweets as tweet}
			<div class="flex justify-between">
				<span class="font-bold">anon says:</span>
				<span class="font-light">{tweet.time}</span>
			</div>
			<div class="mb-2">{tweet.message}</div>
			<div class="flex mb-10">
				<a href="/">
					<span class="font-mono underline">reply</span>
				</a>
				<div class="flex-1"></div>
			</div>
		{/each}
	</div>
</main>