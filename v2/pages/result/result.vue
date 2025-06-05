<template>
	<view class="container">
		<view class="header">
			<view class="title">测试结果分析</view>
			<view class="score-info">
				<view class="score-item">
					<text class="score-label">总分</text>
					<text class="score-value">{{totalScore.toFixed(2)}}分</text>
				</view>
				<view class="score-divider"></view>
				<view class="score-item">
					<text class="score-label">最高分</text>
					<text class="score-value">{{maxScore.toFixed(2)}}分</text>
				</view>
			</view>
		</view>
		
		<view class="result-section" v-if="topType">
			<view class="type-header">
				<view class="type-name">{{topType.name}}</view>
				<view class="score-badge">得分：{{topType.score.toFixed(2)}}分</view>
			</view>
			
			<!-- 改进的下拉列表显示补充内容 -->
			<view class="supplementary-content">
				<view class="dropdown-header" @click="toggleDropdown">
					<view class="dropdown-title">
						<text class="dropdown-icon-title">💡</text>
						<text>治疗建议</text>
					</view>
					<text class="dropdown-icon">{{ isDropdownOpen ? '▼' : '▶' }}</text>
				</view>
				<view class="dropdown-content" v-if="isDropdownOpen">
					<view class="supplementary-item">
						<view class="item-header">
							<text class="item-icon">💊</text>
							<text class="item-title">药物构成</text>
						</view>
						<view class="item-content">{{ getSupplementaryContent(topType.name).medicines }}</view>
					</view>
					<view class="supplementary-item">
						<view class="item-header">
							<text class="item-icon">🎵</text>
							<text class="item-title">五音</text>
						</view>
						<view class="item-content">{{ getSupplementaryContent(topType.name).music }}</view>
					</view>
					<view class="supplementary-item">
						<view class="item-header">
							<text class="item-icon">👂</text>
							<text class="item-title">耳穴</text>
						</view>
						<view class="item-content">{{ getSupplementaryContent(topType.name).acupoints }}</view>
					</view>
				</view>
			</view>
			
			<view class="symptoms-section" v-if="topType && topType.symptoms.length > 0">
				<view class="section-title">
					<text class="section-icon">🔍</text>
					<text>主要症状</text>
				</view>
				<view class="symptoms-list">
					<view class="symptom-tag" v-for="(symptom, index) in topType.symptoms" :key="index">
						{{symptom}}
					</view>
				</view>
			</view>
		</view>
		
		<view class="chart-section">
			<view class="section-title">
				<text class="section-icon">📊</text>
				<text>各证型得分分布</text>
			</view>
			<view class="chart-container">
				<canvas canvas-id="pieChart" class="pie-chart" @click="handleChartClick"></canvas>
			</view>
			<view class="distribution-list">
				<view class="distribution-item" v-for="(result, index) in sortedResults" :key="index" @click="handleTypeClick(result)">
					<view class="distribution-color" :style="{ backgroundColor: getColor(index) }"></view>
					<view class="distribution-info">
						<text class="distribution-name">{{result.name}}</text>
						<text class="distribution-score">{{result.score.toFixed(2)}}分</text>
					</view>
					<text class="distribution-percentage">{{calculatePercentage(result.score)}}%</text>
				</view>
			</view>
		</view>
		
		<view class="action-buttons">
			<button class="action-button retry" @click="retryTest">
				<text class="button-icon">🔄</text>
				<text>重新测试</text>
			</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			results: [],
			topType: null,
			currentType: null, // 当前选中的证型
			maxScore: 0,
			totalScore: 0,
			isDropdownOpen: false,
			supplementaryData: {
				'心脾两虚': {
					medicines: '茯神、黄芪、白术、龙眼肉、酸枣仁等',
					music: '徵音和宫音',
					acupoints: '神门、心、脾'
				},
				'阴虚火旺': {
					medicines: '黄连、朱砂、当归、生地、阿胶等',
					music: '羽音与商音',
					acupoints: '太溪、涌泉、三阴交'
				},
				'肝郁脾虚': {
					medicines: '柴胡、枳实、香附、川芎、陈皮、人参等',
					music: '角音与宫音',
					acupoints: '肝，交感，皮质下'
				},
				'痰热内扰': {
					medicines: '半夏、竹茹、陈皮、枳实、茯苓等',
					music: '商音与宫音',
					acupoints: '神门、心、三焦、脾'
				},
				'肝郁化火': {
					medicines: '柴胡，龙骨，牡蛎，法半夏，桂枝，茯苓等',
					music: '角音',
					acupoints: '太冲、行间、期门'
				}
			}
		}
	},
	computed: {
		displayResults() {
			if (this.currentType) {
				// 如果有选中的证型，只返回该证型的数据
				return [this.results.find(r => r.name === this.currentType)]
			}
			return this.results
		},
		sortedResults() {
			return [...this.results].sort((a, b) => b.score - a.score)
		}
	},
	onLoad(options) {
		if (options.results) {
			this.results = JSON.parse(decodeURIComponent(options.results))
		}
		if (options.topType) {
			this.topType = JSON.parse(decodeURIComponent(options.topType))
		}
		this.maxScore = Math.max(...this.results.map(r => r.score))
		this.totalScore = this.results.reduce((sum, r) => sum + r.score, 0)
		
		this.$nextTick(() => {
			this.drawPieChart()
		})
	},
	methods: {
		handleTypeClick(result) {
			this.currentType = result.name
			this.drawPieChart()
		},
		handleChartClick() {
			// 点击环形图时显示全部
			this.currentType = null
			this.drawPieChart()
		},
		toggleDropdown() {
			this.isDropdownOpen = !this.isDropdownOpen
		},
		getSupplementaryContent(typeName) {
			return this.supplementaryData[typeName] || {
				medicines: '暂无数据',
				music: '暂无数据',
				acupoints: '暂无数据'
			}
		},
		drawPieChart() {
			const ctx = uni.createCanvasContext('pieChart', this)
			const width = 275
			const height = 200
			const centerX = width / 2
			const centerY = height / 2
			const radius = 90
			const innerRadius = 70
			
			// 清空画布
			ctx.clearRect(0, 0, width, height)
			
			// 计算总分和起始角度
			const total = this.totalScore // 始终使用总分来计算比例
			let startAngle = -Math.PI / 2
			
			// 如果是显示单个证型，需要先计算其起始角度
			if (this.currentType) {
				// 计算当前选中证型之前的所有证型分数总和
				const beforeTypes = this.results.slice(
					0, 
					this.results.findIndex(r => r.name === this.currentType)
				)
				const beforeScore = beforeTypes.reduce((sum, item) => sum + item.score, 0)
				// 计算起始角度
				startAngle = -Math.PI / 2 + (2 * Math.PI * beforeScore / total)
			}
			
			// 使用要显示的结果数据
			const data = this.displayResults
			
			// 绘制环形图
			data.forEach((item, index) => {
				const percentage = item.score / total // 使用总分计算真实比例
				const endAngle = startAngle + (2 * Math.PI * percentage)
				
				// 绘制扇形
				ctx.beginPath()
				ctx.arc(centerX, centerY, radius, startAngle, endAngle)
				ctx.lineTo(centerX + innerRadius * Math.cos(endAngle), 
					centerY + innerRadius * Math.sin(endAngle))
				ctx.arc(centerX, centerY, innerRadius, endAngle, startAngle, true)
				ctx.lineTo(centerX + radius * Math.cos(startAngle), 
					centerY + radius * Math.sin(startAngle))
				ctx.closePath()
				
				// 获取颜色时使用原始索引
				const originalIndex = this.results.findIndex(r => r.name === item.name)
				ctx.setFillStyle(this.getColor(originalIndex))
				ctx.fill()
				
				startAngle = endAngle
			})
			
			// 绘制中心文字
			if (data.length === 1) {
				// 单个证型显示时，显示具体分数
				ctx.setTextAlign('center')
				ctx.setFontSize(14)
				ctx.setFillStyle('#333333')
				ctx.fillText(data[0].name, centerX, centerY - 10)
				ctx.setFontSize(20)
				ctx.setFillStyle('#FF5B5B')
				ctx.fillText(`${Math.round(data[0].score / total * 100)}%`, centerX, centerY + 15)
			} else {
				// 显示全部时，显示得分最高的证型和比例
				const topResult = this.sortedResults[0]
				const topPercentage = Math.round((topResult.score / this.totalScore) * 100)
				
				ctx.setTextAlign('center')
				ctx.setFontSize(14)
				ctx.setFillStyle('#333333')
				ctx.fillText(topResult.name, centerX, centerY - 10)
				ctx.setFontSize(20)
				ctx.setFillStyle('#FF5B5B')
				ctx.fillText(`${topPercentage}%`, centerX, centerY + 15)
			}
			
			ctx.draw()
		},
		getColor(index) {
			const colors = ['#4B9EF9', '#FF7B7B', '#FFB66C', '#64DFD6', '#9F7BE9']
			return colors[index % colors.length]
		},
		calculatePercentage(score) {
			return ((score / this.totalScore) * 100).toFixed(1)
		},
		retryTest() {
			// 使用 reLaunch 重新打开首页
			uni.reLaunch({
				url: '/pages/index/index'
			})
		}
	}
}
</script>

<style>
.container {
	padding: 30rpx;
	background-color: #f8f8f8;
	min-height: 100vh;
}

.header {
	background-color: #ffffff;
	border-radius: 20rpx;
	padding: 30rpx;
	margin-bottom: 30rpx;
	box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
}

.title {
	font-size: 36rpx;
	font-weight: bold;
	color: #333333;
	margin-bottom: 20rpx;
	text-align: center;
}

.score-info {
	display: flex;
	justify-content: center;
	align-items: center;
	margin-top: 20rpx;
}

.score-item {
	display: flex;
	flex-direction: column;
	align-items: center;
	padding: 0 30rpx;
}

.score-divider {
	width: 2rpx;
	height: 60rpx;
	background-color: #e0e0e0;
}

.score-label {
	font-size: 28rpx;
	color: #999999;
	margin-bottom: 8rpx;
}

.score-value {
	font-size: 36rpx;
	font-weight: bold;
	color: #4B9EF9;
}

.result-section {
	background-color: #ffffff;
	border-radius: 20rpx;
	padding: 30rpx;
	margin-bottom: 30rpx;
	box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
}

.type-header {
	display: flex;
	flex-direction: column;
	align-items: center;
	margin-bottom: 20rpx;
}

.type-name {
	font-size: 40rpx;
	font-weight: bold;
	color: #333333;
	margin-bottom: 10rpx;
	text-align: center;
}

.score-badge {
	background-color: #4B9EF9;
	color: #ffffff;
	padding: 8rpx 24rpx;
	border-radius: 30rpx;
	font-size: 28rpx;
	font-weight: bold;
}

/* 改进的补充内容下拉列表样式 */
.supplementary-content {
	margin: 20rpx 0;
	border-radius: 16rpx;
	overflow: hidden;
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
}

.dropdown-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 24rpx;
	background: linear-gradient(to right, #f5f5f5, #f0f0f0);
	font-size: 30rpx;
	color: #333333;
	border-radius: 16rpx;
	transition: all 0.3s ease;
}

.dropdown-header:active {
	background: linear-gradient(to right, #e8e8e8, #e0e0e0);
}

.dropdown-title {
	display: flex;
	align-items: center;
}

.dropdown-icon-title {
	margin-right: 10rpx;
	font-size: 32rpx;
}

.dropdown-icon {
	font-size: 24rpx;
	color: #999999;
}

.dropdown-content {
	padding: 24rpx;
	background-color: #ffffff;
	border-bottom-left-radius: 16rpx;
	border-bottom-right-radius: 16rpx;
}

.supplementary-item {
	margin-bottom: 24rpx;
	padding-bottom: 16rpx;
	border-bottom: 1rpx solid #f0f0f0;
}

.supplementary-item:last-child {
	margin-bottom: 0;
	padding-bottom: 0;
	border-bottom: none;
}

.item-header {
	display: flex;
	align-items: center;
	margin-bottom: 12rpx;
}

.item-icon {
	margin-right: 10rpx;
	font-size: 32rpx;
}

.item-title {
	font-size: 30rpx;
	font-weight: bold;
	color: #333333;
}

.item-content {
	font-size: 28rpx;
	color: #666666;
	line-height: 1.6;
	padding-left: 42rpx;
}

.symptoms-section {
	margin-top: 30rpx;
}

.section-title {
	display: flex;
	align-items: center;
	font-size: 32rpx;
	font-weight: bold;
	color: #333333;
	margin-bottom: 20rpx;
}

.section-icon {
	margin-right: 10rpx;
	font-size: 32rpx;
}

.symptoms-list {
	display: flex;
	flex-wrap: wrap;
	gap: 16rpx;
}

.symptom-tag {
	background-color: #f0f0f0;
	color: #666666;
	padding: 12rpx 24rpx;
	border-radius: 30rpx;
	font-size: 26rpx;
	box-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.05);
}

.chart-section {
	background-color: #ffffff;
	border-radius: 20rpx;
	padding: 30rpx;
	margin-bottom: 30rpx;
	box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
}

.chart-container {
	display: flex;
	justify-content: center;
	align-items: center;
	width: 100%;
}

.pie-chart {
	width: 550rpx;
	height: 400rpx;
}

.distribution-list {
	margin-top: 30rpx;
	padding: 0 20rpx;
}

.distribution-item {
	display: flex;
	align-items: center;
	padding: 20rpx;
	margin-bottom: 16rpx;
	background-color: #f8f8f8;
	border-radius: 12rpx;
	transition: all 0.3s ease;
	cursor: pointer;
}

.distribution-item:active {
	background-color: #f0f0f0;
	transform: scale(0.98);
}

.distribution-color {
	width: 24rpx;
	height: 24rpx;
	border-radius: 50%;
	margin-right: 16rpx;
}

.distribution-info {
	flex: 1;
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.distribution-name {
	font-size: 28rpx;
	color: #333333;
}

.distribution-score {
	font-size: 28rpx;
	color: #666666;
	margin-right: 20rpx;
}

.distribution-percentage {
	font-size: 28rpx;
	color: #4B9EF9;
	font-weight: bold;
}

.action-buttons {
	display: flex;
	justify-content: center;
	margin-top: 30rpx;
}

.action-button {
	width: 80%;
	height: 88rpx;
	line-height: 88rpx;
	text-align: center;
	border-radius: 44rpx;
	font-size: 30rpx;
	font-weight: bold;
	display: flex;
	align-items: center;
	justify-content: center;
	box-shadow: 0 4rpx 8rpx rgba(0, 0, 0, 0.1);
}

.button-icon {
	margin-right: 10rpx;
	font-size: 32rpx;
}

.retry {
	background: linear-gradient(to right, #4CAF50, #45a049);
	color: #ffffff;
}
</style> 