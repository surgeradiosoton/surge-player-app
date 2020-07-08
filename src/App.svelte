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


	// rewrite this
	window.generate_audio_stuff = () => {
	var context = new AudioContext();
	var src, analyser, renderMethods = {};
	let iOS=false;
	let audio = new Audio();
	audio.crossOrigin = "anonymous";
	audio.src = stream_url;
	audio.load();
	src = context.createMediaElementSource(audio);
	analyser = context.createAnalyser();
	// onClick of first interaction on page before I need the sounds
	// audio.play();
	var canvas = document.getElementById("canvas");
	canvas.width = window.innerWidth;
	canvas.height = window.innerHeight;
	var ctx = canvas.getContext("2d");
	// document.getElementById("volMax").value = audio.volume*10;

	src.connect(analyser);
	analyser.connect(context.destination);

	analyser.fftSize = 256;
	var bufferLength = analyser.frequencyBinCount;
	console.log(bufferLength);
	var dataArray = new Uint8Array(bufferLength);
	var WIDTH = canvas.width;
	var HEIGHT = canvas.height;
	// var barWidth = (WIDTH / bufferLength) * 2.5;
	var barWidth=16,barHeight =200, maxBarNum = 24, barSpacing = 0;
	var x = 0;

	renderMethods.sphere = function() {
	// buggy sphere method. roll out in late 2020.
	requestAnimationFrame(renderMethods.sphere);

	x = 0, y=0;
	ctx.clearRect(0, 0, WIDTH, HEIGHT);
	// ctx.fillStyle = "#000";
	// ctx.fillRect(0, 0, WIDTH*2, HEIGHT);
	analyser.getByteFrequencyData(dataArray);

	// ctx.fillStyle = "#000";
	// ctx.fillRect(0, 0, WIDTH, HEIGHT);
	var cx = WIDTH / 2;
	var cy = HEIGHT / 2;
	var radius = 320;
	var maxBarNum = Math.floor((radius * 2 * Math.PI) / (barWidth + barSpacing));
	var slicedPercent = Math.floor((maxBarNum * 25) / 100);
	var barNum = maxBarNum - slicedPercent;
	var freqJump = Math.floor(dataArray.length / maxBarNum);

	for (var i = 0; i < bufferLength; i++) {
	  amplitude = dataArray[i * freqJump];
	  alfa = (i * 2 * Math.PI ) / maxBarNum;
	  beta = (3 * 45 - barWidth) * Math.PI / 180;
	  barHeight = dataArray[i];
	  
	  // var r = barHeight + (25 * (i/bufferLength));
	  // var g = 250 * (i/bufferLength);
	  // var b = 50;

	  // ctx.fillStyle = "rgb(" + r + "," + g + "," + b + ")";


	  var h = 216;
	  var s = barHeight + (25 * (i/bufferLength));
	  var l = 40;

	  ctx.fillStyle = "hsl(" + h + "," + s + "%," + l + "%)";


	  // ctx.fillRect(x, HEIGHT - barHeight, barWidth, barHeight);
	  x = 0;
	  y = radius - (amplitude / 8 - barHeight);
	  w = barWidth;
	  h = amplitude / 6 + barHeight;

	  ctx.save();
	  ctx.translate(cx + 2, cy + 2);
	  ctx.rotate(alfa - beta);
	  ctx.fillRect(x, y, w, h);
	  ctx.restore();
	  x += barWidth + 1;
	}
	}

	renderMethods.bars = function ()  {
	requestAnimationFrame(renderMethods.bars);

	x = 0;

	analyser.getByteFrequencyData(dataArray);

	ctx.clearRect(0, 0, WIDTH, HEIGHT);
	// ctx.fillStyle = "#000";
	// ctx.fillRect(0, 0, WIDTH*2, HEIGHT);

	for (var i = 0; i < bufferLength; i++) {
	  barHeight = dataArray[i];

	  var h = 216;
	  var s = barHeight + (25 * (i/bufferLength));
	  var l = 40;

	  ctx.fillStyle = "hsl(" + h + "," + s + "%," + l + "%)";
	  ctx.fillRect(x, HEIGHT - barHeight, barWidth, barHeight);

	  x += barWidth + 1;
	}
	}
	// audio.volume = document.getElementById("vol-slider-control").value/10;
	audio.play();
	renderMethods.bars();
	if (iOS==true){ document.getElementById("apple_are_bellends").classList.add("hidden");
	document.getElementById("pp").classList.remove("hidden");}

	window.onclose = function() {
	audio.close();
	}
	window.onreload = function() {
	audio.close();
	}
	}

	onMount(async() => {
		generate_audio_stuff();
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