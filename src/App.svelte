<script>

	import {fade,blur} from "svelte/transition"
	import {onMount, afterUpdate} from "svelte"

	import TiMediaPlay from 'svelte-icons/ti/TiMediaPlay.svelte'
	import TiMediaPause from 'svelte-icons/ti/TiMediaPause.svelte'
	import TiVolume from 'svelte-icons/ti/TiVolume.svelte'

	let canvas;
	let device_type = 'pc'; // we'll add an iOS flag at the end
	let ctime = new Date();
	let stream_url = "https://cors-anywhere.herokuapp.com/http://stream.susu.org:8000/surge-live-high-mp3";
	let slogans = [
		'Your student soundtrack.',
		'The sounds of Southampton.',
		'Playing your music.'
	]
	let current_slogan = 0;
	$: current_time = ctime.getTime() && get_time();

	// create audio stream
	let context = new AudioContext(), audio = new Audio(), src, analyser;
	audio.crossOrigin = "anonymous";
	audio.src = stream_url;

	let get_time = () => {
		return `${ctime.getHours().toString().padStart(2, '0')}:${ctime.getMinutes().toString().padStart(2, '0')}:${ctime.getSeconds().toString().padStart(2, '0')}`;
	}

	onMount(async() => {
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
		audio.play();
		console.log(audio);
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
			audio.play()
	}

	// this is quicker than polling the audio event itself
	let music_toggle_check = true;

	$: visible_slogan = slogans[current_slogan];

</script>

<canvas id="canvas" bind:this={canvas}></canvas>
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

<style>

	.header-controls > div > button {
		/* svelte-icons requires you to do this */
		width:  48px;
		height: 48px;
	}

	.header-controls > div {    
		padding: 12px;
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