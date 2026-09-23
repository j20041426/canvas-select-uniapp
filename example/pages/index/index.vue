<template>
	<view class="page">
		<view class="canvas-wrap">
			<jrm-canvas-select
				style="width: 100%; height: 710rpx"
				url="/static/bg.jpg"
				@inited="onInited"
			></jrm-canvas-select>
		</view>

		<view class="status">
			<view class="status-item">
				<text class="status-k">坐标</text>
				<text class="status-v">{{ positionText }}</text>
			</view>
			<view class="status-item">
				<text class="status-k">缩放</text>
				<text class="status-v">{{ scalePercent }}%</text>
			</view>
			<view class="status-item">
				<text class="status-k">标注</text>
				<text class="status-v">{{ shapeCount }}</text>
			</view>
			<view class="status-item">
				<text class="status-k">模式</text>
				<text class="status-v">{{ modeName }}</text>
			</view>
			<view class="status-item wide">
				<text class="status-k">选中</text>
				<text class="status-v">{{ activeText }}</text>
			</view>
		</view>

		<view class="section">
			<view class="section-title">🎨 创建工具</view>
			<view class="grid">
				<view
					v-for="item in TOOLS"
					:key="item.type"
					class="btn"
					:class="{ active: createType === item.type }"
					@click="setCreateType(item.type)"
				>
					{{ item.name }}
				</view>
			</view>
		</view>

		<view class="section">
			<view class="section-title">🔍 视图控制</view>
			<view class="grid">
				<view class="btn" @click="zoomIn">放大</view>
				<view class="btn" @click="zoomOut">缩小</view>
				<view class="btn" @click="fitZoom">适配画布</view>
				<view
					class="btn"
					:class="{ active: focusMode }"
					@click="toggleFocusMode"
					>专注模式</view
				>
			</view>
			<view class="row">
				<text class="row-label">显示十字坐标线</text>
				<switch :checked="showCross" @change="onCrossChange" />
			</view>
		</view>

		<view class="section">
			<view class="section-title">⚙️ 模式设置</view>
			<view class="row">
				<text class="row-label">只读模式</text>
				<switch :checked="readonlyMode" @change="onReadonlyChange" />
			</view>
			<view class="row">
				<text class="row-label">锁定画布</text>
				<switch :checked="lockMode" @change="onLockChange" />
			</view>
			<view class="row">
				<text class="row-label">矩形旋转控制</text>
				<switch
					:checked="showRotation"
					@change="onShowRotationChange"
				/>
			</view>
		</view>

		<view class="section">
			<view class="section-title">🎨 样式设置</view>
			<view class="row-label">边框颜色</view>
			<view class="palette">
				<view
					v-for="c in STROKE_COLORS"
					:key="c"
					class="swatch"
					:class="{ active: strokeStyle === c }"
					:style="{ background: c }"
					@click="onStrokeColor(c)"
				></view>
			</view>
			<view class="row-label">填充颜色</view>
			<view class="palette">
				<view
					v-for="c in FILL_COLORS"
					:key="c"
					class="swatch"
					:class="{ active: fillStyle === c }"
					:style="{ background: c }"
					@click="onFillColor(c)"
				></view>
			</view>
			<view class="row-label">边框宽度：{{ lineWidth }}</view>
			<slider
				:value="lineWidth"
				:min="1"
				:max="10"
				show-value
				@change="onLineWidthChange"
			/>
			<view class="row-label">控制点大小：{{ ctrlRadius }}</view>
			<slider
				:value="ctrlRadius"
				:min="2"
				:max="10"
				show-value
				@change="onCtrlRadiusChange"
			/>
		</view>

		<view class="section">
			<view class="section-title">🖌️ 画笔设置</view>
			<view class="row-label">画笔颜色</view>
			<view class="palette">
				<view
					v-for="c in BRUSH_COLORS"
					:key="c"
					class="swatch"
					:class="{ active: brushColor === c }"
					:style="{ background: c }"
					@click="onBrushColor(c)"
				></view>
			</view>
			<view class="row-label">画笔大小：{{ brushSize }}</view>
			<slider
				:value="brushSize"
				:min="1"
				:max="50"
				show-value
				@change="onBrushSizeChange"
			/>
			<view class="row-label">橡皮擦大小：{{ eraserSize }}</view>
			<slider
				:value="eraserSize"
				:min="5"
				:max="100"
				show-value
				@change="onEraserSizeChange"
			/>
		</view>

		<view class="section">
			<view class="section-title">🏷️ 标签设置</view>
			<view class="row">
				<text class="row-label">标签字体</text>
				<picker
					mode="selector"
					:range="FONT_LIST"
					:value="labelFontIndex"
					@change="onFontChange"
				>
					<view class="picker-value">{{
						FONT_LIST[labelFontIndex]
					}}</view>
				</picker>
			</view>
			<view class="row-label">标签背景色</view>
			<view class="palette">
				<view
					v-for="c in FILL_COLORS"
					:key="c"
					class="swatch"
					:class="{ active: labelFillStyle === c }"
					:style="{ background: c }"
					@click="onLabelFillColor(c)"
				></view>
			</view>
			<view class="row-label">标签文字颜色</view>
			<view class="palette">
				<view
					v-for="c in STROKE_COLORS"
					:key="c"
					class="swatch"
					:class="{ active: textFillStyle === c }"
					:style="{ background: c }"
					@click="onTextColor(c)"
				></view>
			</view>
			<view class="row">
				<text class="row-label">隐藏标签</text>
				<switch :checked="hideLabel" @change="onHideLabelChange" />
			</view>
			<view class="row">
				<text class="row-label">标签显示在上方</text>
				<switch :checked="labelUp" @change="onLabelUpChange" />
			</view>
		</view>

		<view class="section">
			<view class="section-title">🔲 网格设置</view>
			<view class="row">
				<text class="row-label">行</text>
				<input class="num-input" type="number" v-model="gridRow" />
			</view>
			<view class="row">
				<text class="row-label">列</text>
				<input class="num-input" type="number" v-model="gridCol" />
			</view>
			<view class="grid">
				<view class="btn" @click="applyGrid">应用到选中网格</view>
			</view>
		</view>

		<view class="section">
			<view class="section-title">💾 数据操作</view>
			<view class="grid">
				<view class="btn" @click="loadSampleData">加载示例数据</view>
				<view class="btn" @click="clearAllData">清空所有数据</view>
				<view class="btn" @click="exportData">导出数据(复制)</view>
				<view class="btn" @click="importData">导入数据(粘贴)</view>
			</view>
			<view class="grid">
				<view class="btn danger" @click="deleteSelected">删除选中</view>
				<view class="btn" @click="clearBrush">清除画笔</view>
			</view>
		</view>

		<view class="section">
			<view class="section-title">🖼️ 图片操作</view>
			<view class="row">
				<text class="row-label">图片URL</text>
				<input
					class="text-input"
					v-model="imageUrl"
					placeholder="输入图片URL"
				/>
			</view>
			<view class="grid">
				<view class="btn" @click="changeImage">更换图片</view>
				<view class="btn" @click="chooseLocalImage">选择本地图片</view>
			</view>
			<view class="grid">
				<view class="btn" @click="exportImage">{{
					exporting ? "导出中..." : "导出图片"
				}}</view>
				<view class="btn" @click="saveToAlbum">保存到相册</view>
			</view>
			<view class="output" v-if="lastExportPath"
				>导出路径：{{ lastExportPath }}</view
			>
		</view>

		<view class="section">
			<view class="section-title">📊 标注数据输出</view>
			<view class="output">{{ dataOutput }}</view>
		</view>

		<view class="section">
			<view class="section-title">📝 事件日志</view>
			<view class="log">
				<view v-if="!eventLog.length" class="log-item"
					>系统已初始化，等待操作...</view
				>
				<view v-for="(item, i) in eventLog" :key="i" class="log-item">
					<text class="log-time">{{ item.time }}</text>
					<text> - {{ item.event }}</text>
					<text v-if="item.detail" class="log-detail">{{
						item.detail
					}}</text>
				</view>
			</view>
		</view>

		<view class="section help">
			<view class="section-title">💡 操作提示（触摸屏）</view>
			<view class="help-item">矩形：单指按住拖动</view>
			<view class="help-item">多边形：单指点击加点，点击起始点闭合</view>
			<view class="help-item">点标注：单指单击</view>
			<view class="help-item">折线：单指点击加点，双击完成</view>
			<view class="help-item">圆形：单指按住拖动确定半径</view>
			<view class="help-item"
				>网格：本面板设置行列，双击单元格选中/取消</view
			>
			<view class="help-item">画笔 / 橡皮擦：单指按住拖动</view>
			<view class="help-item">画布：单指拖动画布，双指捏合缩放</view>
			<view class="help-item">移动形状：先点选，再单指拖动</view>
		</view>
	</view>
</template>

<script setup>
import { ref, computed, onUnmounted } from "vue";

const TOOLS = [
	{ type: 0, name: "选择模式" },
	{ type: 1, name: "矩形" },
	{ type: 2, name: "多边形" },
	{ type: 3, name: "点标注" },
	{ type: 4, name: "折线" },
	{ type: 5, name: "圆形" },
	{ type: 6, name: "网格" },
	{ type: 7, name: "画笔" },
	{ type: 8, name: "橡皮擦" },
];
const SHAPE_NAMES = {
	0: "选择模式",
	1: "矩形",
	2: "多边形",
	3: "点",
	4: "折线",
	5: "圆形",
	6: "网格",
	7: "画笔",
	8: "橡皮擦",
};
const STROKE_COLORS = [
	"#00ff00",
	"#ff0000",
	"#0000ff",
	"#ffff00",
	"#ff00ff",
	"#00ffff",
	"#ffffff",
	"#000000",
];
const FILL_COLORS = [
	"#ff0000",
	"#00ff00",
	"#0000ff",
	"#ffff00",
	"#ff00ff",
	"#00ffff",
	"#ffffff",
	"#000000",
];
const BRUSH_COLORS = [
	"#ff0000",
	"#00ff00",
	"#0000ff",
	"#ffff00",
	"#ff00ff",
	"#00ffff",
	"#000000",
];
const FONT_LIST = [
	"10px sans-serif",
	"12px sans-serif",
	"14px sans-serif",
	"10px Arial",
	"12px Arial",
];

/** CanvasSelect 实例保持非响应式，避免 Vue 代理 canvas 上下文 */
let instance = null;
let sampleLoaded = false;
let lastRefresh = 0;
let positionTimer = null;

const createType = ref(0);
const positionText = ref("0, 0");
const scalePercent = ref(100);
const shapeCount = ref(0);
const activeText = ref("无");
const focusMode = ref(false);
const showCross = ref(false);
const readonlyMode = ref(false);
const lockMode = ref(false);
const showRotation = ref(false);
const strokeStyle = ref("#00ff00");
const fillStyle = ref("#0000ff");
const lineWidth = ref(1);
const ctrlRadius = ref(3);
const brushColor = ref("#ff0000");
const brushSize = ref(20);
const eraserSize = ref(20);
const labelFontIndex = ref(0);
const labelFillStyle = ref("#ffffff");
const textFillStyle = ref("#000000");
const hideLabel = ref(false);
const labelUp = ref(false);
const gridRow = ref(3);
const gridCol = ref(2);
const imageUrl = ref("/static/bg.jpg");
const dataOutput = ref("暂无标注数据");
const eventLog = ref([]);
const lastExportPath = ref("");
const exporting = ref(false);

const modeName = computed(() => SHAPE_NAMES[createType.value] || "选择模式");

function hexToRgba(hex, alpha) {
	const h = String(hex).replace("#", "");
	if (h.length !== 6) return hex;
	const r = parseInt(h.slice(0, 2), 16);
	const g = parseInt(h.slice(2, 4), 16);
	const b = parseInt(h.slice(4, 6), 16);
	return `rgba(${r}, ${g}, ${b}, ${alpha})`;
}

function addLog(event, detail) {
	const top = eventLog.value[0];
	if (top && top.event === event && top.detail === detail) return;
	const time = new Date().toLocaleTimeString();
	eventLog.value.unshift({ time, event, detail });
	if (eventLog.value.length > 50)
		eventLog.value = eventLog.value.slice(0, 50);
}

function refreshUI(force = false) {
	if (!instance) return;
	const now = Date.now();
	if (!force && now - lastRefresh < 200) return;
	lastRefresh = now;
	scalePercent.value = Math.round(instance.scale * 100);
	shapeCount.value = instance.dataset.length;
	const active = instance.activeShape;
	activeText.value =
		active && active.type
			? SHAPE_NAMES[active.type] +
				(active.label ? `「${active.label}」` : "")
			: "无";
	if (instance.dataset.length) {
		dataOutput.value = JSON.stringify(instance.dataset, null, 2);
	} else {
		dataOutput.value = "暂无标注数据";
	}
}

function buildSampleData() {
	const W = instance.IMAGE_ORIGIN_WIDTH || 800;
	const H = instance.IMAGE_ORIGIN_HEIGHT || 600;
	const X = (r) => Math.round(W * r);
	const Y = (r) => Math.round(H * r);
	const R = Math.round(Math.min(W, H) * 0.1);
	return [
		{
			label: "矩形",
			coor: [
				[X(0.08), Y(0.1)],
				[X(0.36), Y(0.32)],
			],
			type: 1,
		},
		{
			label: "多边形",
			coor: [
				[X(0.42), Y(0.12)],
				[X(0.55), Y(0.08)],
				[X(0.62), Y(0.22)],
				[X(0.54), Y(0.34)],
				[X(0.43), Y(0.28)],
			],
			type: 2,
		},
		{ label: "点", coor: [X(0.72), Y(0.2)], type: 3 },
		{
			label: "折线",
			coor: [
				[X(0.1), Y(0.45)],
				[X(0.25), Y(0.55)],
				[X(0.4), Y(0.42)],
				[X(0.55), Y(0.52)],
			],
			type: 4,
		},
		{ label: "圆形", coor: [X(0.75), Y(0.5)], radius: R, type: 5 },
		{
			label: "网格",
			coor: [
				[X(0.08), Y(0.65)],
				[X(0.45), Y(0.92)],
			],
			type: 6,
			row: 3,
			col: 2,
			selected: [1, 3],
		},
		{
			label: "画笔",
			coor: [
				[X(0.55), Y(0.68)],
				[X(0.6), Y(0.72)],
				[X(0.65), Y(0.7)],
				[X(0.7), Y(0.76)],
				[X(0.75), Y(0.72)],
			],
			type: 7,
			brushSize: 6,
			brushStokeStyle: "#ff0000",
		},
		{
			label: "橡皮擦",
			coor: [
				[X(0.62), Y(0.66)],
				[X(0.66), Y(0.78)],
			],
			type: 8,
			eraserSize: 20,
		},
	];
}

function loadSampleOnce() {
	if (sampleLoaded || !instance) return;
	sampleLoaded = true;
	const data = buildSampleData();
	instance.setData(data);
	addLog("加载示例数据", `${data.length} 条`);
}

/** jrm-canvas-select 初始化完成，回调参数即 CanvasSelect 实例 */
function onInited(inst) {
	if (!inst) return;
	instance = inst;

	instance.isMobile = true;
	// 移动端无右键菜单，网格行列改由面板输入框设置
	instance.gridMenuEnable = false;
	instance.createType = 0;
	instance.strokeStyle = strokeStyle.value;
	instance.fillStyle = hexToRgba(fillStyle.value, 0.25);
	instance.lineWidth = lineWidth.value;
	instance.ctrlRadius = ctrlRadius.value;
	instance.brushStokeStyle = brushColor.value;
	instance.brushSize = brushSize.value;
	instance.eraserSize = eraserSize.value;
	instance.labelFont = FONT_LIST[labelFontIndex.value];
	instance.labelFillStyle = labelFillStyle.value;
	instance.textFillStyle = textFillStyle.value;

	instance.on("load", () => {
		addLog("图片加载完成", "");
		// handleLoad 先 emit('load') 再赋值尺寸并 fitZoom，故延迟读取
		setTimeout(loadSampleOnce, 50);
	});
	instance.on("warn", (msg) => {
		addLog("警告", String(msg));
	});
	instance.on("add", (info) => {
		addLog(
			"添加标注",
			`${SHAPE_NAMES[info.type] || "未知"}${info.label ? `「${info.label}」` : ""}`,
		);
		refreshUI(true);
	});
	instance.on("delete", (info) => {
		addLog(
			"删除标注",
			`${SHAPE_NAMES[info.type] || "未知"}${info.label ? `「${info.label}」` : ""}`,
		);
		refreshUI(true);
	});
	instance.on("select", (info) => {
		addLog(
			info ? "选中标注" : "取消选中",
			info ? SHAPE_NAMES[info.type] || "" : "",
		);
		refreshUI(true);
	});
	instance.on("updated", () => refreshUI());

	// 图片加载失败兜底，避免示例数据永不出现
	setTimeout(loadSampleOnce, 3000);

	// 组件未暴露触摸回调，坐标读数改为轮询实例上的 position
	positionTimer = setInterval(() => {
		if (!instance) return;
		positionText.value = `${instance.position[0]}, ${instance.position[1]}`;
	}, 100);

	refreshUI(true);
	addLog("初始化完成", `画布 ${instance.WIDTH}×${instance.HEIGHT}`);
}

function setCreateType(type) {
	createType.value = type;
	if (instance) instance.createType = type;
	addLog("切换创建模式", SHAPE_NAMES[type]);
}

function zoomIn() {
	if (instance) instance.setScale(true);
}
function zoomOut() {
	if (instance) instance.setScale(false);
}
function fitZoom() {
	if (instance) instance.fitZoom();
	addLog("适配画布", "");
}
function toggleFocusMode() {
	if (!instance) return;
	focusMode.value = !instance.focusMode;
	instance.setFocusMode(focusMode.value);
	refreshUI(true);
	addLog("切换专注模式", focusMode.value ? "开启" : "关闭");
}

function onCrossChange(e) {
	showCross.value = e.detail.value;
	if (instance) {
		instance.showCross = showCross.value;
		instance.update();
	}
}
function onReadonlyChange(e) {
	readonlyMode.value = e.detail.value;
	if (instance) instance.readonly = readonlyMode.value;
}
function onLockChange(e) {
	lockMode.value = e.detail.value;
	if (instance) instance.lock = lockMode.value;
}
function onShowRotationChange(e) {
	showRotation.value = e.detail.value;
	if (instance) {
		instance.showRotation = showRotation.value;
		instance.dataset.forEach((s) => {
			if (s.type === 1) s.showRotation = showRotation.value;
		});
		instance.update();
	}
}

function applyStyle() {
	if (!instance) return;
	instance.strokeStyle = strokeStyle.value;
	instance.fillStyle = hexToRgba(fillStyle.value, 0.25);
	instance.lineWidth = Number(lineWidth.value);
	instance.ctrlRadius = Number(ctrlRadius.value);
	instance.dataset.forEach((s) => {
		if (s.type === 7 || s.type === 8) return;
		s.strokeStyle = instance.strokeStyle;
		s.fillStyle = instance.fillStyle;
		s.lineWidth = instance.lineWidth;
	});
	instance.update();
	refreshUI(true);
}
function onStrokeColor(c) {
	strokeStyle.value = c;
	applyStyle();
}
function onFillColor(c) {
	fillStyle.value = c;
	applyStyle();
}
function onLineWidthChange(e) {
	lineWidth.value = Number(e.detail.value);
	applyStyle();
}
function onCtrlRadiusChange(e) {
	ctrlRadius.value = Number(e.detail.value);
	applyStyle();
}

function onBrushColor(c) {
	brushColor.value = c;
	if (instance) instance.brushStokeStyle = c;
}
function onBrushSizeChange(e) {
	brushSize.value = Number(e.detail.value);
	if (instance) instance.brushSize = brushSize.value;
}
function onEraserSizeChange(e) {
	eraserSize.value = Number(e.detail.value);
	if (instance) instance.eraserSize = eraserSize.value;
}

function applyLabelStyle() {
	if (!instance) return;
	instance.labelFont = FONT_LIST[labelFontIndex.value];
	instance.labelFillStyle = labelFillStyle.value;
	instance.textFillStyle = textFillStyle.value;
	instance.hideLabel = hideLabel.value;
	instance.labelUp = labelUp.value;
	instance.update();
}
function onFontChange(e) {
	labelFontIndex.value = Number(e.detail.value);
	applyLabelStyle();
}
function onLabelFillColor(c) {
	labelFillStyle.value = c;
	applyLabelStyle();
}
function onTextColor(c) {
	textFillStyle.value = c;
	applyLabelStyle();
}
function onHideLabelChange(e) {
	hideLabel.value = e.detail.value;
	applyLabelStyle();
}
function onLabelUpChange(e) {
	labelUp.value = e.detail.value;
	applyLabelStyle();
}

function applyGrid() {
	if (!instance) return;
	const active = instance.activeShape;
	if (!active || active.type !== 6) {
		uni.showToast({ title: "请先选中一个网格", icon: "none" });
		return;
	}
	const row = Math.max(1, Math.floor(Number(gridRow.value) || 1));
	const col = Math.max(1, Math.floor(Number(gridCol.value) || 1));
	active.row = row;
	active.col = col;
	instance.update();
	refreshUI(true);
	addLog("设置网格行列", `${row} 行 ${col} 列`);
}

function loadSampleData() {
	if (!instance) return;
	instance.setData(buildSampleData());
	// setData 内部经 setTimeout 延迟写入 dataset，等其完成后再刷新展示
	setTimeout(() => refreshUI(true), 60);
	uni.showToast({ title: "示例数据已加载", icon: "none" });
}
function clearAllData() {
	if (!instance) return;
	instance.setData([]);
	setTimeout(() => refreshUI(true), 60);
	addLog("清空所有数据", "");
}
function exportData() {
	if (!instance) return;
	uni.setClipboardData({
		data: JSON.stringify(instance.dataset),
		success: () => {
			addLog("导出数据", `${instance.dataset.length} 条已复制到剪贴板`);
			uni.showToast({ title: "已复制到剪贴板", icon: "none" });
		},
		fail: () => uni.showToast({ title: "复制失败", icon: "none" }),
	});
}
function importData() {
	if (!instance) return;
	uni.getClipboardData({
		success: (res) => {
			try {
				const data = JSON.parse(res.data);
				if (!Array.isArray(data)) throw new Error("not array");
				instance.setData(data);
				setTimeout(() => refreshUI(true), 60);
				addLog("导入数据", `${data.length} 条`);
				uni.showToast({ title: "导入成功", icon: "none" });
			} catch (err) {
				uni.showToast({
					title: "剪贴板内容不是有效标注数据",
					icon: "none",
				});
			}
		},
		fail: () => uni.showToast({ title: "读取剪贴板失败", icon: "none" }),
	});
}
function deleteSelected() {
	if (!instance) return;
	const active = instance.activeShape;
	if (active && active.index !== undefined) {
		instance.deleteByIndex(active.index);
		refreshUI(true);
	} else {
		uni.showToast({ title: "请先选中要删除的标注", icon: "none" });
	}
}
function clearBrush() {
	if (!instance) return;
	instance.clearBrush();
	refreshUI(true);
	addLog("清除画笔数据", "");
}

function changeImage() {
	if (!instance) return;
	if (!imageUrl.value) {
		uni.showToast({ title: "请输入图片URL", icon: "none" });
		return;
	}
	instance.setImage(imageUrl.value);
	addLog("更换图片", imageUrl.value);
}
function chooseLocalImage() {
	uni.chooseImage({
		count: 1,
		sourceType: ["album", "camera"],
		success: (res) => {
			const src = res.tempFilePaths[0];
			if (instance) instance.setImage(src);
			addLog("选择本地图片", src);
		},
	});
}

async function exportImage() {
	if (!instance || exporting.value) return;
	exporting.value = true;
	try {
		// update() 内部有 10ms 防抖，先重绘并让出一帧，避免导出到旧画面
		instance.update();
		await new Promise((resolve) => setTimeout(resolve, 100));
		const tempFilePath = await instance.exportImage({
			fileType: "jpg",
			quality: 0.9,
		});
		lastExportPath.value = tempFilePath;
		addLog("导出图片", tempFilePath);
		uni.previewImage({ urls: [tempFilePath] });
	} catch (err) {
		addLog("导出图片失败", String(err && err.errMsg ? err.errMsg : err));
		uni.showToast({ title: "导出图片失败", icon: "none" });
	} finally {
		exporting.value = false;
	}
}

function saveToAlbum() {
	if (!lastExportPath.value) {
		uni.showToast({ title: "请先导出图片", icon: "none" });
		return;
	}
	// #ifdef H5
	uni.showToast({ title: "H5 请长按预览图保存", icon: "none" });
	addLog("保存到相册", "H5 环境请长按预览图保存");
	return;
	// #endif
	uni.saveImageToPhotosAlbum({
		filePath: lastExportPath.value,
		success: () => {
			addLog("保存到相册", lastExportPath.value);
			uni.showToast({ title: "已保存到相册", icon: "none" });
		},
		fail: (err) => {
			addLog("保存到相册失败", String(err && err.errMsg ? err.errMsg : err));
			uni.showToast({ title: "保存失败，请检查相册权限", icon: "none" });
		},
	});
}

onUnmounted(() => {
	if (positionTimer) {
		clearInterval(positionTimer);
		positionTimer = null;
	}
});
</script>

<style>
.page {
	padding: 20rpx;
	background: #f5f6f8;
	min-height: 100vh;
}

.canvas-wrap {
	background: #ffffff;
	border-radius: 16rpx;
	overflow: hidden;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.06);
}

.status {
	display: flex;
	flex-wrap: wrap;
	margin-top: 20rpx;
	padding: 16rpx;
	background: #343a40;
	border-radius: 16rpx;
}

.status-item {
	width: 50%;
	display: flex;
	align-items: center;
	padding: 6rpx 0;
	font-size: 24rpx;
	color: #ffffff;
}

.status-item.wide {
	width: 100%;
}

.status-k {
	color: #adb5bd;
	margin-right: 10rpx;
}

.status-v {
	font-weight: 600;
}

.section {
	margin-top: 20rpx;
	padding: 20rpx;
	background: #ffffff;
	border-radius: 16rpx;
	box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.06);
}

.section-title {
	font-size: 28rpx;
	font-weight: 600;
	color: #495057;
	margin-bottom: 16rpx;
}

.grid {
	display: flex;
	flex-wrap: wrap;
}

.btn {
	flex: 0 0 auto;
	min-width: 150rpx;
	margin: 0 12rpx 12rpx 0;
	padding: 14rpx 16rpx;
	border: 2rpx solid #e9ecef;
	border-radius: 10rpx;
	background: #ffffff;
	font-size: 24rpx;
	color: #495057;
	text-align: center;
}

.btn.active {
	background: #667eea;
	border-color: #667eea;
	color: #ffffff;
}

.btn.danger {
	border-color: #dc3545;
	color: #dc3545;
}

.row {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 10rpx 0;
}

.row-label {
	font-size: 26rpx;
	color: #495057;
	margin: 10rpx 0 6rpx;
}

.palette {
	display: flex;
	flex-wrap: wrap;
	margin-bottom: 10rpx;
}

.swatch {
	width: 56rpx;
	height: 56rpx;
	margin: 0 12rpx 12rpx 0;
	border-radius: 10rpx;
	border: 4rpx solid #e9ecef;
}

.swatch.active {
	border-color: #667eea;
}

.picker-value {
	font-size: 24rpx;
	color: #667eea;
	padding: 8rpx 16rpx;
	border: 2rpx solid #e9ecef;
	border-radius: 10rpx;
}

.num-input,
.text-input {
	min-width: 160rpx;
	padding: 10rpx 16rpx;
	border: 2rpx solid #e9ecef;
	border-radius: 10rpx;
	font-size: 24rpx;
	background: #f8f9fa;
}

.text-input {
	flex: 1;
	margin-left: 16rpx;
}

.output {
	max-height: 400rpx;
	overflow-y: auto;
	padding: 16rpx;
	background: #f8f9fa;
	border-radius: 10rpx;
	font-family: monospace;
	font-size: 20rpx;
	color: #495057;
	word-break: break-all;
}

.log {
	max-height: 400rpx;
	overflow-y: auto;
	padding: 16rpx;
	background: #fff3cd;
	border-radius: 10rpx;
}

.log-item {
	font-size: 22rpx;
	color: #664d03;
	padding: 6rpx 0;
	border-bottom: 2rpx solid #ffeaa7;
	word-break: break-all;
}

.log-item:last-child {
	border-bottom: none;
}

.log-time {
	color: #997404;
}

.log-detail {
	color: #997404;
	margin-left: 8rpx;
}

.help {
	background: #e3f2fd;
}

.help .section-title {
	color: #1976d2;
}

.help-item {
	font-size: 24rpx;
	color: #424242;
	padding: 6rpx 0;
}
</style>
