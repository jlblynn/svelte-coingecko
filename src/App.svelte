<script>
	import { onMount } from "svelte";
  import { scaleLinear } from "d3-scale";

	const url = "https://api.coingecko.com/api/v3/";
	let connected = false;
	let data;

	async function checkConnection () {
		const endpoint = url + "ping";
		const response = await fetch(endpoint);
		if (response) {
			connected = true;
			data = await response.json();
			return;
		} else {
			console.log("No connection");
		}
	}

	let coin = "bitcoin";
	let coinData;
	let price = {};
	let atl;
	let ath;
	let mcap;
	let totalVol;
	async function getPrice() {
		const endpoint = url + `coins/${coin}`;
		const response = await fetch(endpoint);
		if (response) {
			console.log("price query success")
			coinData = await response.json();
			price = coinData.market_data.current_price.usd;
			atl = coinData.market_data.atl.usd;
			ath = coinData.market_data.ath.usd;
			mcap = coinData.market_data.market_cap.usd;
			totalVol = coinData.market_data.total_volume.usd;
			return;
		} else {
			console.log("Price query error");
		}
	}

	const points = [
		{ x: 1000, y: 44204 },
		{ x: 1100, y: 44017 },
		{ x: 1200, y: 44173 },
		{ x: 1300, y: 44360 },
		{ x: 1400, y: 44357 },
		{ x: 1500, y: 44296 },
		{ x: 1600, y: 44000 },
		{ x: 1700, y: 43909 },
		{ x: 1800, y: 43851 }
	];

	const yTicks = [40000, 41000, 42000, 43000, 44000];
	const xTicks = [1000, 1100, 1200, 1300, 1400, 1500, 1600, 1700];
	const padding = { top: 50, right: 15, bottom: 20, left: 25 };

	let width = 500;
	let height = 200;

	$: xScale = scaleLinear()
		.domain([minX, maxX])
		.range([padding.left, width - padding.right]);

	$: yScale = scaleLinear()
		.domain([Math.min.apply(null, yTicks), Math.max.apply(null, yTicks)])
		.range([height - padding.bottom, padding.top]);

	$: minX = points[0].x;
	$: maxX = points[points.length - 1].x;
	$: path = `M${points.map(p => `${xScale(p.x)},${yScale(p.y)}`).join('L')}`;
	$: area = `${path}L${xScale(maxX)},${yScale(39000)}L${xScale(minX)},${yScale(39000)}Z`;

	function formatMobile (tick) {
		return "'" + tick.toString().slice(-2);
	}
  

  onMount(async function () {
    await checkConnection();
		await getPrice();
  });

  
</script>

<main>

	<h1>Live Bitcoin Price</h1>

	<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
		<svg>
			<!-- y axis -->
			<g class="axis y-axis" transform="translate(0, {padding.top})">
				{#each yTicks as tick}
					<g class="tick tick-{tick}" transform="translate(0, {yScale(tick) - padding.bottom})">
						<line x2="100%"></line>
						<text y="-4">{tick} {tick === 8 ? ' million sq km' : ''}</text>
					</g>
				{/each}
			</g>
	
			<!-- x axis -->
			<g class="axis x-axis">
				{#each xTicks as tick}
					<g class="tick tick-{ tick }" transform="translate({xScale(tick)},{height})">
						<line y1="-{height}" y2="-{padding.bottom}" x1="0" x2="0"></line>
						<text y="-2">{width > 380 ? tick : formatMobile(tick)}</text>
					</g>
				{/each}
			</g>
	
			<!-- data -->
			<path class="path-area" d={area}></path>
			<path class="path-line" d={path}></path>
		</svg>
	</div>
	
	<p>Connected: {connected}</p>
	<p>Price: ${price}</p>
	<p>ATL: ${atl}</p>
	<p>ATL: ${ath}</p>
	<p>Market cap: ${mcap}</p>
	<p>Total Volume: ${totalVol}</p>

</main>

<style>
	.chart, h2, p {
		width: 100%;
		max-width: 500px;
		margin-left: auto;
		margin-right: auto;
	}

	svg {
		position: relative;
		width: 100%;
		height: 200px;
		overflow: visible;
	}

	.tick {
		font-size: .725em;
		font-weight: 200;
	}

	.tick line {
		stroke: #aaa;
		stroke-dasharray: 2;
	}

	.tick text {
		fill: #666;
		text-anchor: start;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.x-axis .tick text {
		text-anchor: middle;
	}

	.path-line {
		fill: none;
		stroke: rgb(0,100,100);
		stroke-linejoin: round;
		stroke-linecap: round;
		stroke-width: 2;
	}

	.path-area {
		fill: rgba(0,100,100,0.2);
	}


	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>