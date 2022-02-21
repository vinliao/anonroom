<script>
	import * as secp from "@noble/secp256k1";
	import { format } from 'timeago.js';
	import Tweet from "./Tweet.svelte";
	import Nav from "./Nav.svelte";
	import Dots from "./Dots.svelte";

	// public and private key of production should be separate from the dev one
	// maybe environment variable?
	// const privateKey = "6ba903b7888191180a0959a6d286b9d0719d33a47395c519ba107470412d2069";
	// const publicKey = "8565b1a5a63ae21689b80eadd46f6493a3ed393494bb19d0854823a757d8f35f";

	// dummy keypair for development
	const privateKey = "b86e8ca23e9594b0d6d78291c23b3fc20e749c464925bdff82fc57ec95e48d0e";
	const publicKey = "6fe50495793e11d3d866b20f45cfe26293a099a25f0533ca031ea16828b10444";
	let inputMessage;
	let toBeReplied;
	let formInput;

	function toHexString(byteArray) {
		return Array.prototype.map
			.call(byteArray, function (byte) {
				return ("0" + (byte & 0xff).toString(16)).slice(-2);
			})
			.join("");
	}

	async function createEvent() {
		let tags = [];
		if(toBeReplied) {
			tags.push(["e", toBeReplied.id]);
		}

		const content = inputMessage;
		const unixTime = Math.floor(Date.now() / 1000);
		const data = [0, publicKey, unixTime, 1, tags, content];

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
			tags: tags,
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

		// if there's replied
		// potential bug: if the replied event is outside the tweets array
		let replied;
		if(event.tags.length > 0) {
			let repliedId = event.tags[0][1];
			let repliedTweet = tweets.find(o => o.id === repliedId);
			replied = {"message": repliedTweet.message, "time": repliedTweet.time};
		}

		let tweet = {"message": event.content, "time": format(timestamp), "id": event.id, "replied": replied};
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
		toBeReplied = null;
	}

	function fillReplyData(id, message, time) {
		toBeReplied = {message: message, time: time, id: id};
		// todo: this works, but still buggy, not always working
		formInput.focus();
		document.body.scrollIntoView(); // back to top
	}

	function deleteReplyData() {
		toBeReplied = null;
	}
</script>

<main class="max-w-lg mx-auto">
	<div class="flex flex-col items-stretch justify-between shrink-0">
		<Nav/>

		<form on:submit|preventDefault="{submit}">
			<div class="flex flex-col"> 
				<textarea bind:value="{inputMessage}" bind:this={formInput} rows="5" placeholder="What's on your mind, anon?" class="focus:ring-inset focus:ring-red-600 focus:border-slate-500"></textarea>
				<div class="flex">
					<div class="flex-1"></div>
					<button class="bg-red-800 text-white py-2 px-3 font-mono">send</button>
				</div>
			</div>
		</form>

		{#if toBeReplied}
			<div class="p-3 border border-slate-500">
				<div class="flex justify-between">
					<span class="font-bold">anon says:</span>
					<span class="font-light">{toBeReplied.time}</span>
				</div>
				<div>{toBeReplied.message}</div>
				<div class="flex">
					<div class="flex-1"></div>
					<button class="font-mono underline" on:click="{deleteReplyData}">cancel</button>
				</div>
			</div>
		{/if}

		<Dots />

		{#if tweets.length > 0}
			<div class="border-b border-slate-500"></div>
		{/if}

		{#each tweets as tweet}
			<div class="border-x border-b border-slate-500 p-3">
				<!-- tweet component is for view, not logic -->
				<Tweet message="{tweet.message}" time="{tweet.time}" replied="{tweet.replied}"/>
				<div class="flex">
					<button class="font-mono underline" on:click="{fillReplyData(tweet.id, tweet.message, tweet.time)}">reply</button>
					<div class="flex-1"></div>
				</div>
			</div>
		{/each}
	</div>
</main>