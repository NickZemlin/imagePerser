<template>
	<div class="main">
		<canvas id="viewport"></canvas>
		<div class="imgs-wrap">
			<span v-for="img in imgArray">
				<Transition name="fade">
					<p
						v-show="img.brightness"
						class="stats"
						:style="{ color: img.fontColor }"
					>
						{{ 'brightness: ' + img.brightness }} <br />
						{{ 'runtime: ' + img.runtime + 'ms' }}
					</p>
				</Transition>
				<img
					:src="img.src"
					alt=""
					:id="img.index.toString()"
					@mouseenter="getBrightnes('enter', img.index)"
					@mouseleave="getBrightnes('leave', img.index)"
				/>
			</span>
		</div>
	</div>
</template>

<script setup lang="ts">
	import { ref } from 'vue';

	const randomImgs = ref<string[]>([
		'https://media.istockphoto.com/photos/random-multicolored-spheres-computer-generated-abstract-form-of-large-picture-id1295274245?k=20&m=1295274245&s=612x612&w=0&h=3wSeid9PHT1A9K3L_TawCyvmU6rLK0pDGAp4JuErvBg=',
		'https://static1.squarespace.com/static/5ac589eb8ab722aa77be2eeb/5b479b056d2a73336ed80ffe/5b50d290562fa7d2dabe2955/1532023444871/random_interactive_share.jpg?2',
		'https://thumbs.dreamstime.com/b/heaven-cloud-sky-sunny-bright-future-heaven-cloud-sky-sunny-bright-future-wealth-fortune-day-concept-123441100.jpg',
	]);

	class specificImg {
		public constructor(img: string, index: number) {
			this.src = img;
			this.index = index;
			this.once = true;
		}
		src!: string;
		index!: number;
		brightness!: number;
		runtime!: number;
		once!: boolean;
		fontColor!: string;
	}

	function generateImgs(): Array<specificImg> {
		let temp: Array<specificImg> = [];
		for (let i in randomImgs.value) {
			temp.push(new specificImg(randomImgs.value[i], Number(i)));
		}
		return temp;
	}

	const imgArray = ref<specificImg[]>(generateImgs());

	function getBrightnes(intent: string, index: number): void {
		if (intent === 'enter') {
			let startTime = new Date().getTime();
			let imgEl: any = document.getElementById(index.toString());
			let img = document.createElement('img');
			img.crossOrigin = 'anonymous';
			img.src = imgEl.src;
			img.id = `temp${index}`;
			img.style.display = 'none';
			document.body.appendChild(img);
			let colorSum = 0;

			img.onload = function () {
				let canvas = document.createElement('canvas');
				canvas.width = img.width / 3;
				canvas.height = 100;

				let ctx = canvas.getContext('2d');
				if (ctx) {
					ctx.drawImage(img, 0, 0);

					let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
					let data = imageData.data;
					let r, g, b, avg;

					for (var x = 0, len = data.length; x < len; x += 4) {
						r = data[x];
						g = data[x + 1];
						b = data[x + 2];

						avg = Math.floor((r + g + b) / 3);
						colorSum += avg;
					}

					let brightness = Math.floor(
						colorSum / (canvas.width * canvas.height)
					);
					imgArray.value[index].brightness = brightness;
					let endTime = new Date().getTime();
					if (imgArray.value[index].once) {
						imgArray.value[index].runtime = endTime - startTime;
						imgArray.value[index].once = false;
					}
					imgArray.value[index].fontColor =
						brightness > 255 / 2 ? 'black' : 'white';
				}
			};
		} else {
			imgArray.value[index].brightness = 0;
			let tempImg = document.getElementById(`temp${index}`);
			if (tempImg) {
				tempImg.outerHTML = '';
			}
		}
	}
</script>

<style>
	* {
		padding: 0;
		margin: 0;
		box-sizing: border-box;
	}
	.stats {
		position: absolute;
		font-size: 2em;
	}
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		color: #2c3e50;
		display: flex;
		width: 100vw;
		height: 100vh;
		align-items: center;
		justify-content: center;
		background-color: azure;
	}
	.imgs-wrap {
		display: flex;
		gap: 20px;
	}
	img {
		height: 300px;
		border-radius: 5px;
		box-shadow: 0.5px 0.6px 2.2px rgba(0, 0, 0, 0.13),
			1.2px 1.3px 5.3px rgba(0, 0, 0, 0.093),
			2.3px 2.5px 10px rgba(0, 0, 0, 0.077),
			4px 4.5px 17.9px rgba(0, 0, 0, 0.065),
			7.5px 8.4px 33.4px rgba(0, 0, 0, 0.053),
			18px 20px 80px rgba(0, 0, 0, 0.037);
	}

	.fade-enter-from {
		opacity: 0;
	}
	.fade-enter-to {
		opacity: 1;
	}
	.fade-enter-active {
		transition: all 1s ease;
	}
	.fade-leave-from {
		opacity: 1;
	}
	.fade-leave-to {
		opacity: 0;
	}
	.fade-leave-active {
		transition: all 0.3s ease;
	}
</style>
