<script>
	export let name;

	import {fade,blur} from "svelte/transition"
	import {onMount, afterUpdate} from "svelte"

	let ctime = new Date();
	let stream_url = "https://cors-anywhere.herokuapp.com/http://stream.susu.org:8000/surge-live-high-mp3";
	$: current_time = ctime.getTime() && get_time();

	let get_time = () => {
		return `${ctime.getHours().toString().padStart(2, '0')}:${ctime.getMinutes().toString().padStart(2, '0')}:${ctime.getSeconds().toString().padStart(2, '0')}`;
	}

	setInterval(() => {
		ctime = new Date();
	}, 1000);

	import generate_audio_stuff from './og_viz.js'
	onMount(async() => {
		generate_audio_stuff(stream_url);
	})

</script>

<canvas id="canvas"></canvas>
<main in:fade>
	<div class="container" id="container">
		<div>
			<h1>{current_time}</h1>
			<h2>SURGE RADIO</h2>
			<h4>Your student soundtrack.</h4>
		</div>
		
	</div>
 
</main>

<style>
	html, body {
		position: relative;
		width: 100%;
		height: 100%;

		color: white;
	}

	.hidden {
		display:none;
	}

	#fixWithTime {
		font-family: "Source Code Pro",Monaco,Courier,monospace;
		text-align: center;
		font-weight: 100;
	}

	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-family: "Source Code Pro",Monaco,Courier,monospace;
		text-align: center;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>