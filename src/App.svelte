<script>

	import {fade,blur} from "svelte/transition"
	import {onMount, afterUpdate} from "svelte"
	import { detect } from "detect-browser"

	import TiMediaPlay from 'svelte-icons/ti/TiMediaPlay.svelte'
	import TiMediaPause from 'svelte-icons/ti/TiMediaPause.svelte'
	import TiVolume from 'svelte-icons/ti/TiVolume.svelte'
	import TiVolumeMute from 'svelte-icons/ti/TiVolumeMute.svelte'
	
	const browser = detect();

	let canvas;
	let ctime = new Date();
	let stream_url = "https://cors-anywhere.herokuapp.com/http://stream.susu.org:8000/surge-live-high-mp3";
	let slogans = [
		'Your student soundtrack.',
		'The sounds of Southampton.',
		'Playing your music.'
	]
	let current_slogan = 0, current_volume = 5;
	$: current_time = ctime.getTime() && get_time();

	// this is quicker than polling the audio event itself
	let music_toggle_check = true;

	// set up audio data
	let audio = new Audio(), context, src, analyser;

	// browser compatibility checking
	let player_fallback = false;
	if (11==(browser.version.split(".")[0]))
		player_fallback = true;

	$: visible_slogan = slogans[current_slogan];


	let get_time = () => {
		return `${ctime.getHours().toString().padStart(2, '0')}:${ctime.getMinutes().toString().padStart(2, '0')}:${ctime.getSeconds().toString().padStart(2, '0')}`;
	}

	onMount(async() => {

		// create audio stream
		switch (browser && browser.name) {
			case 'safari':
				audio = document.querySelector("audio");
				context = new webkitAudioContext();
				if (browser.os == "iOS") {

				}
				break;
			default:
				context = new AudioContext();
				audio.crossOrigin = "anonymous";
				audio.src = stream_url;
				break;
		};

		audio.load();
		// canvas stuff
		canvas.width = window.innerWidth;
		canvas.height = window.innerHeight;
		let ctx = canvas.getContext("2d");

		// create audio analysis stuff
		src = context.createMediaElementSource(audio);
		analyser = context.createAnalyser();
		src.connect(analyser);
		analyser.connect(context.destination);
		analyser.fftSize = 256;
		var buffer_len = analyser.frequencyBinCount;
		var buffer_contents = new Uint8Array(buffer_len);

		// render sub function
		let render = () => {
			let bar_width = 16,
				bar_height = 500, 
				x = 0;
			requestAnimationFrame(render);
			analyser.getByteFrequencyData(buffer_contents);
			ctx.clearRect(0, 0, canvas.width, canvas.height);
			for (var i = 0; i < buffer_len; i++) {
				bar_height = buffer_contents[i]*2;
				var h = 216;
				var s = (bar_height*0.7) + (25 * (i/buffer_len));
				var l = 50+(25 * (i/buffer_len));
				ctx.fillStyle = "hsl(" + h + "," + s + "%," + l + "%)";
				ctx.fillRect(x, canvas.height - bar_height, bar_width, bar_height);
				x += bar_width + 1;
			}
		}
		music_control_volume();
		if (browser.name !== "safari") {
			audio.play();
		} else {
			console.log("safari is irritating, you have to press the button");
		}
		render();
		window.onclose = function() {
			audio.close();
		}
		window.onreload = function() {
			audio.close();
		}
	});

	setInterval(() => {
		ctime = new Date();
		// console.log(audio.readyState);
	}, 1000);

	setInterval(() => {	
		current_slogan = Math.floor(Math.random()*slogans.length)
	}, 7000);

	let music_control_toggle = () => {
		music_toggle_check = !music_toggle_check;
		if (!audio.paused)
			audio.pause();
		else
			audio.play();
	}

	let music_control_volume = () => {
		audio.volume = current_volume/10;
	}

	let music_control_mute = () => {
		// there is a more svelte-y way to do this using $:
		// but for now this'll do!
		if (audio.volume == 0) {
			audio.volume = 0.5;
			current_volume = 5;
		}
		else {
			audio.volume = 0;
			current_volume = 0;
		}
	}

	let safari_play_audio = () => {
		document.querySelector("audio").src = stream_url;
		document.querySelector("audio").load();
		document.querySelector("audio").play();
	};

</script>

<canvas id="canvas" bind:this={canvas}></canvas>
{#if player_fallback == false}
{#if browser.name == "safari"}
	<audio src="emerge.mp3" crossorigin="anonymous" preload="true" autoplay></audio>
	<button on:click={safari_play_audio}>Play audio</button>
{/if}
<div class="header-controls">
	{#if audio.readyState == 4}
		<div in:fade>
			<button on:click={music_control_toggle}>
				{#if music_toggle_check==true}
					<TiMediaPause />
				{:else}
					<TiMediaPlay />
				{/if}
			</button>
			<button on:click={music_control_mute}>
				{#if current_volume == 0}
					<TiVolumeMute />
				{:else}
					<TiVolume />
				{/if}
			</button>
			<input type="range" bind:value={current_volume} on:input={music_control_volume} min="0" max="10">
		</div>
	{/if}
</div>
<main in:fade>
	<div class="container" id="container">
		<div>
			<h1>{current_time}</h1>
			<h2>SURGE RADIO</h2>
			<h3 transition:fade>{visible_slogan}</h3>
		</div>
		
	</div>
</main>
{:else}
<main>
	<h1>{current_time}</h1>
	<h2>SURGE RADIO</h2>
	<p>Your browser does not support the cool visualiser we've built, so here's our TuneIn.</p>
	<iframe src="https://tunein.com/embed/player/s37346/?background=dark&scheme=006DB"scrolling="no" frameborder="no"></iframe>

</main>
{/if}

<style>

	iframe {
		height:100px;
	}

	.header-controls > div > button {
		/* svelte-icons requires you to do this */
		width:  48px;
		height: 48px;
		-webkit-appearance:none;
		appearance:none;
		text-align: left;
		background: white;
	}

	.header-controls > div > input {
		/* svelte-icons requires you to do this */
		height: 48px;
		-webkit-appearance:none;
		appearance:none;
		background: white;
	}

	.header-controls > div {    
		padding: 12px;    
		display: flex;
	}

	.header-controls > div > * {       
		margin-right: 6px;
		transition: 120ms ease;
		opacity:0.2;
	}

	.header-controls > div > *:hover {     
		opacity:0.5; 
	}

	.header-controls {
		height: 60px;
	}

	html, body {
		position: relative;
		width: 100%;
		height: 100%;

		color: white;
	}
	
	h2 {
		font-size:34pt;
		margin-top: 0;
	}

	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #003edd;
		text-transform: uppercase;
		font-size: 4em;
		font-family: "Source Code Pro",Monaco,Courier,monospace;
		text-align: center;
		font-weight: 200;
		margin-bottom: 0;
		letter-spacing: 1.5px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>