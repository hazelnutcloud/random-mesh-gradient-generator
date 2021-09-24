<script>
	import { onDestroy, onMount, tick } from "svelte";
	import { interpolate } from "d3-interpolate";
	import { cubicOut } from "svelte/easing";

	//color array declaration
	const hexColors = [
		"ffadad",
		"ffd6a5",
		"fdffb6",
		"caffbf",
		"9bf6ff",
		"a0c4ff",
		"bdb2ff",
		"ffc6ff",
		"fffffc",
	];
	function hexToRGB(hex) {
		const RGB = hex.match(/[0-9a-f]{1,2}/gi);
		return RGB.map((value) => parseInt(value, 16));
	}
	const colors = hexColors.map(hexToRGB);

	function randomNumber(size) {
		return Math.floor(Math.random() * size);
	}

	function pickRandomColor(colors) {
		return colors[randomNumber(colors.length)];
	}

	function generateRadialGradient(x, y, color) {
		const result = `radial-gradient(at ${x}% ${y}%, rgb(${color[0]}, ${color[1]}, ${color[2]}) 0, transparent 50%)`;
		return result;
	}

	function generateBackgroundStyle(positions, colors) {
		let style = '';
		for (let i = 0; i < colors.length - 1; i++) {
			style = style.concat(`${generateRadialGradient(positions.x[i], positions.y[i], colors[i])},`)
		}
		style = style.concat(`${generateRadialGradient(positions.x[colors.length-1], positions.y[colors.length-1], colors[colors.length-1])}`)
		return style;
	}
	
	let colorCount = 10;
	//positions and colors
	let prevColors = [];
	let prevPos = {
		x: [],
		y: []
	};
	for (let i = 0; i < colorCount; i++) {
		prevColors.push(pickRandomColor(colors));
		prevPos.x.push(randomNumber(101));
		prevPos.y.push(randomNumber(101));
	}
	let nextColors = [...prevColors];
	let nextPos = {...prevPos};
	
	$: settings = {
		startColors: prevColors,
		endColors: nextColors,
		startPositions: prevPos,
		endPositions: nextPos
	}

	let style;
	//update background to next gradient
	function changeBackground(ele, positions, colors) {
		style = generateBackgroundStyle(positions, colors);
		ele.style.background = style;
	}

	//handle click
	async function handleClick() {
		prevColors = [...nextColors];
		nextColors = [];
		prevPos = {...nextPos}
		nextPos = {
			x: [],
			y: []
		}
		for (let i = 0; i < colorCount; i++) {
			nextColors.push(pickRandomColor(colors));
			nextPos.x.push(randomNumber(100));
			nextPos.y.push(randomNumber(100));
		}
		await tick();
		changeBackground(background, nextPos, nextColors);
	}

	async function copy() {
		await navigator.clipboard.writeText(style);
		console.log('copied');
	}


	let background;
	let loop;
	onMount(() => {
		changeBackground(background, nextPos, nextColors);
		// loop = setInterval(handleClick, 2001);
	});

	// onDestroy(() => {
	// 	clearInterval(loop);
	// })

	function smoothcolor(
		node,
		{ easing = cubicOut, duration = 1000, startColors, endColors, startPositions, endPositions }
	) {
		const colors = interpolate(startColors, endColors);
		const positions = interpolate(startPositions, endPositions);
		return {
			easing,
			duration,
			css: (t) => {
				return 'background: ' + generateBackgroundStyle(positions(t), colors(t));
			},
		};
	}
</script>

{#key settings}
	<div
		class="center"
		bind:this={background}
		in:smoothcolor={settings}
	>
		
		<button on:click={() => colorCount -= 1} title="decrease colors on screen">-</button>
		<p><span class="hidden-mobile">Number of colors on screen: </span>{colorCount}</p>
		<button on:click={() => colorCount += 1} title="increase colors on screen">+</button>
		<button on:click={handleClick}>New Gradient</button>
		<button on:click={copy} title="copy css code"><i class="far fa-clipboard"></i></button>
	</div>
{/key}

<style>
	div {
		width: 100%;
		height: 100%;
		min-width: 380px;
		background-color: #FFFFFC;
	}
	.center {
		display: flex;
		justify-content: center;
		align-items: flex-end;
	}
	button {
		border-radius: 6px;
		padding: 0.5em 1.5em;
		/* From https://css.glass */
		background: rgba(255, 255, 255, 0.2);
		border-radius: 16px;
		box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
		backdrop-filter: blur(5px);
		-webkit-backdrop-filter: blur(5px);
		border: 2px solid rgba(255, 255, 255, 0.3);
		margin: 1em;
	}
	p {
		margin-bottom: 1.6em;
	}
	@media screen and (max-width: 690px) {
		button {
			margin: 0.2em;
			padding: 0.5em 1em;
		}
		.hidden-mobile {
			display: none;
		}
		p {
			margin: 0 1em 1em 1em; 
		}
	}
</style>
