<template>
	<view class="jrm-canvas-select-wrap">
		<canvas
			class="canvas"
			canvas-id="canvas-select"
			disable-scroll
			@touchstart="onTouchStart"
			@touchmove="onTouchMove"
			@touchend="onTouchEnd"
			@touchcancel="onTouchEnd"
		></canvas>
		<canvas
			class="canvas-offscreen"
			canvas-id="canvas-select-offscreen"
			disable-scroll
		></canvas>
	</view>
</template>

<script>
import CanvasSelect from "../../../../../src/index";

export default {
	props: {
		url: {
			type: String,
			default: "",
		},
	},
	emit: ["inited"],
	data() {
		return {
			instance: null,
			touch: { down: null, move: null, up: null },
		};
	},
	methods: {
		initCanvas() {
			uni.createSelectorQuery()
				.select(".jrm-canvas-select-wrap>.canvas")
				.boundingClientRect((data) => {
					if (!data || !data.width || !data.height) {
						setTimeout(this.initCanvas, 100);
						return;
					}
					this.instance = new CanvasSelect(
						{
							canvasId: "canvas-select",
							offscreenCanvasId: "canvas-select-offscreen",
							width: data.width,
							height: data.height,
							left: data.left,
							top: data.top,
						},
						this.url,
					);

					this.touch.down = this.instance.adapterEvent(
						this.instance.handleMouseDown,
					);
					this.touch.move = this.instance.adapterEvent(
						this.instance.handleMouseMove,
					);
					this.touch.up = this.instance.adapterEvent(
						this.instance.handleMouseUp,
					);

					this.$emit("inited", this.instance);
				})
				.exec();
		},
		onTouchStart(e) {
			this.touch.down(e);
		},
		onTouchMove(e) {
			this.touch.move(e);
		},
		onTouchEnd(e) {
			this.touch.up(e);
		},
	},
	mounted() {
		this.initCanvas();
	},
};
</script>

<style>
.jrm-canvas-select-wrap {
	position: relative;
	width: 100%;
	height: 100%;
}
.jrm-canvas-select-wrap .canvas {
	width: 100%;
	height: 100%;
}
.jrm-canvas-select-wrap .canvas-offscreen {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	z-index: -1;
	transform: translateZ(-1px);
}
</style>
