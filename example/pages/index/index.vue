<template>
	<view class="content">
		<image class="logo" src="/static/logo.png" @click="openImage"></image>
		<view class="text-area">
			<text class="title" @click="confirm">{{title}}</text>
		</view>
		<view class="container">
			<canvas class="canvas" canvas-id="canvas-select"
				@touchstart="(e) => cs.adapterEvent(cs.handleMouseDown)(e)"
				@touchmove="(e) => cs.adapterEvent(cs.handleMouseMove)(e)"
				@touchend="(e) => cs.adapterEvent(cs.handleMouseUp)(e)"
				@touchcancel="(e) => cs.adapterEvent(cs.handleMouseUp)(e)" />
		</view>
	</view>
</template>

<script setup>
import { ref, shallowRef } from 'vue';
import CanvasSelect from '../../../src/index';

const title = ref('uniapp');
const cs = shallowRef({});
const init = (imageSrc) => {
	const ctx = uni.createCanvasContext('canvas-select');
	const query = uni.createSelectorQuery();
	const dpr = uni.getDeviceInfo()
	query.select('.canvas').boundingClientRect((data) => {
		cs.value = new CanvasSelect({ctx, width: data.width, height: data.height, left: data.left, top: data.top}, imageSrc);
		cs.value.on('add', info => console.log('添加形状:' + info));
		cs.value.createType = 1;
		cs.value.strokeStyle = '#ffffff'
		cs.value.alpha = false
		cs.value.setData([
			{
				label: "rectangle",
				labelFillStyle: "#f00",
				textFillStyle: "#fff",
				coor: [
					[184, 183],
					[575, 538],
				], // 必需
				type: 1, // 必需 (1: 矩形)
			},
		])
	}).exec()
}

const openImage = () => {
	uni.chooseImage({
		count: 1,
		sourceType: ['album'],
		success: res => {
			init(res.tempFilePaths[0]);
		}
	})
}

const confirm = () => {
	const annotateArr = cs.value.dataset.map((v) => {
	    return [
	      1,
	      (v.coor[0][0] + v.coor[1][0]) / 2 / cs.value.IMAGE_ORIGIN_WIDTH,
	      (v.coor[0][1] + v.coor[1][1]) / 2 / cs.value.IMAGE_ORIGIN_HEIGHT,
	      (v.coor[1][0] - v.coor[0][0]) / cs.value.IMAGE_ORIGIN_WIDTH,
	      (v.coor[1][1] - v.coor[0][1]) / cs.value.IMAGE_ORIGIN_HEIGHT,
	    ]
	  })
	  console.log(annotateArr)
		uni.canvasToTempFilePath({
			canvasId: 'canvas-select',
			fileType: 'jpg',
			quality: 0.9,
			success: res => {
				console.log(res)
			}
		})
}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}

	.container {
		width: 100vw;
		height: calc(100vh - 300rpx);
	}
	.canvas {
		width: 100%;
		height: 100%;
	}
</style>
