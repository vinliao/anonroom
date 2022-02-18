<script>
	import * as secp from "@noble/secp256k1";

	const privateKey = "c248d1ad9b7c8994c7ca4a25076a6cbd620f8b5fe8006d489c0db2e4bce2d5c4";
	const publicKey = "dd20c14acca7bc078cd35d86aabf3bad33ae5633fad22d56108a99f8e6179ecf";
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

	// websocket stuff
	// let url = "ws://localhost:8080";
	let url = "wss://nostr-pub.wellorder.net";
	let socket = new WebSocket(url);

	socket.onopen = function (event) {
		console.log(`Connected to ${url}`)
		socket.send(JSON.stringify(["REQ", "foobar", {"authors": [publicKey]}]));
	};

	// take data and parse here
	socket.onmessage = function (event) {
		console.log(event.data);
	}

	let texts = ["This is first text", 
		"This is second text",
		"This is third text this is third text this is third text this is third text",
		"This is fourth text",
	];

	async function submit() {
		// submit to relay here
		let event = await createEvent();
		socket.send(JSON.stringify(["EVENT", event]));
		texts.unshift(inputMessage);
		// force re-render
		texts = texts;
		inputMessage = "";
	}
</script>

<main style="max-width: 576px; margin: auto;">
	<div class="d-flex flex-column align-items-stretch justify-content-between flex-shrink-0 bg-white">
		<div class="d-flex justify-content-between mb-3">
			<a href="/" class="d-flex p-3 link-dark text-decoration-none">
				<span class="fw-bold">anonroom</span>
			</a>

			<!-- todo: point to github repo -->
			<a href="https://github.com" class="d-flex p-3 link-dark text-decoration-none">
				<span class="fw-light">about</span>
			</a>
		</div>

		<form on:submit|preventDefault="{submit}">
			<div class="input-group"> 
				<input type="text" class="form-control" bind:value="{inputMessage}" placeholder="What's on your mind, anon?"> 
				<button class="btn btn-outline-secondary" type="submit">Send</button>
			</div>
		</form>

		<div class="list-group overflow-auto">
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
	</div>
</main>