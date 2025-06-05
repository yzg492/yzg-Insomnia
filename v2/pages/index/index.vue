<template>
	<view class="container">
		<!-- 左侧证型分类导航 -->
		<view class="type-nav">
			<view 
				v-for="(type, index) in types" 
				:key="index"
				:class="['type-item', currentType === index ? 'active' : '']"
				@tap="switchType(index)"
			>
				<text class="type-text">{{type.name}}</text>
			</view>
		</view>
		
		<!-- 右侧症状列表 -->
		<scroll-view class="symptom-list" scroll-y>
			<view class="list-header">
				<text class="header-title">请选择您的症状程度</text>
				<text class="header-subtitle">点亮星星评分</text>
			</view>
			<view class="symptom-card" v-for="(symptom, index) in currentSymptoms" :key="index">
				<view class="symptom-name">{{symptom.name}}</view>
				<view class="symptom-score">
					<view class="stars">
						<text 
							v-for="n in 6" 
							:key="n"
							:class="['star', symptom.score >= (n * symptom.maxScore / 5) ? 'active' : '']"
							@tap="updateScore(symptom, n)"
						>★</text>
					</view>
					<text class="score-text">{{symptom.score}}分</text>
				</view>
			</view>
		</scroll-view>
		
		<!-- 底部按钮 -->
		<view class="bottom-button">
			<button type="primary" @tap="calculateResult">开始分析</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			currentType: 0,
			types: [
				{
					name: '心脾两虚',
					symptoms: [
						{ name: '健忘', score: 0, maxScore: 9 },
						{ name: '大便不成形', score: 0, maxScore: 8 },
						{ name: '食欲不佳', score: 0, maxScore: 7 },
						{ name: '多梦易醒', score: 0, maxScore: 6 },
						{ name: '日间疲乏', score: 0, maxScore: 5 },
						{ name: '心悸', score: 0, maxScore: 7 }
					]
				},
				{
					name: '肝郁化火',
					symptoms: [
						{ name: '口苦', score: 0, maxScore: 10 },
						{ name: '便秘', score: 0, maxScore: 9 },
						{ name: '头痛（胀痛）', score: 0, maxScore: 8 },
						{ name: '情绪急躁', score: 0, maxScore: 7 },
						{ name: '入睡困难', score: 0, maxScore: 6 }
					]
				},
				{
					name: '肝郁脾虚',
					symptoms: [
						{ name: '胁胀', score: 0, maxScore: 9 },
						{ name: '反酸', score: 0, maxScore: 8 },
						{ name: '腹胀', score: 0, maxScore: 7 },
						{ name: '大便溏泄', score: 0, maxScore: 7 },
						{ name: '情绪抑郁', score: 0, maxScore: 6 }
					]
				},
				{
					name: '痰热内扰',
					symptoms: [
						{ name: '痰黄稠/痰黏', score: 0, maxScore: 10 },
						{ name: '口苦', score: 0, maxScore: 6 },
						{ name: '便秘', score: 0, maxScore: 6 },
						{ name: '头昏沉重', score: 0, maxScore: 5 },
						{ name: '胸闷', score: 0, maxScore: 8 },
						{ name: '小便短赤', score: 0, maxScore: 6 }
					]
				},
				{
					name: '阴虚火旺',
					symptoms: [
						{ name: '潮热盗汗', score: 0, maxScore: 10 },
						{ name: '五心烦热', score: 0, maxScore: 9 },
						{ name: '口干咽燥', score: 0, maxScore: 8 },
						{ name: '眠浅易醒', score: 0, maxScore: 6 },
						{ name: '眩晕（眼花）', score: 0, maxScore: 7 }
					]
				}
			]
		}
	},
	computed: {
		currentSymptoms() {
			return this.types[this.currentType].symptoms
		}
	},
	methods: {
		switchType(index) {
			this.currentType = index
		},
		updateScore(symptom, starCount) {
			// 计算每颗星对应的分数（五分之一的总分）
			const scorePerStar = symptom.maxScore / 5
			const newScore = Number((starCount * scorePerStar).toFixed(2))
			
			if (symptom.score === newScore) {
				symptom.score = 0 // 如果点击当前分数，则清零
			} else {
				symptom.score = newScore
			}
		},
		calculateResult() {
			// 计算每个证型的总分和症状得分情况
			const results = this.types.map(type => ({
				name: type.name,
				score: type.symptoms.reduce((sum, symptom) => sum + symptom.score, 0),
				symptoms: type.symptoms.filter(s => s.score > 0).map(s => s.name)
			}))
			
			// 找出得分最高的证型
			const maxScore = Math.max(...results.map(r => r.score))
			const topType = results.find(r => r.score === maxScore)
			
			// 跳转到结果页
			uni.navigateTo({
				url: `/pages/result/result?results=${encodeURIComponent(JSON.stringify(results))}&topType=${encodeURIComponent(JSON.stringify(topType))}`
			})
		}
	}
}
</script>

<style>
.container {
	display: flex;
	height: 100vh;
	background-color: #F8FAFC;
}

.type-nav {
	width: 200rpx;
	background-color: #fff;
	height: 100%;
	box-shadow: 2rpx 0 10rpx rgba(0,0,0,0.05);
}

.type-item {
	padding: 40rpx 20rpx;
	text-align: center;
	position: relative;
	transition: all 0.3s ease;
}

.type-item.active {
	background-color: #F0F7FF;
	color: #4B9EF9;
}

.type-item.active::after {
	content: '';
	position: absolute;
	left: 0;
	top: 50%;
	transform: translateY(-50%);
	width: 6rpx;
	height: 40rpx;
	background-color: #4B9EF9;
	border-radius: 0 3rpx 3rpx 0;
}

.type-text {
	font-size: 28rpx;
	line-height: 1.4;
}

.symptom-list {
	flex: 1;
	height: calc(100vh - 120rpx);
	padding: 30rpx;
}

.list-header {
	margin-bottom: 40rpx;
	padding: 20rpx 0;
}

.header-title {
	display: block;
	font-size: 32rpx;
	color: #333;
	font-weight: 500;
	margin-bottom: 10rpx;
}

.header-subtitle {
	font-size: 24rpx;
	color: #999;
}

.symptom-card {
	background-color: #fff;
	border-radius: 16rpx;
	padding: 30rpx;
	margin-bottom: 24rpx;
	box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.04);
	transition: all 0.3s ease;
}

.symptom-card:active {
	transform: scale(0.98);
}

.symptom-name {
	font-size: 30rpx;
	color: #333;
	margin-bottom: 24rpx;
}

.symptom-score {
	display: flex;
	align-items: center;
}

.stars {
	display: flex;
	margin-right: 20rpx;
}

.star {
	color: #E1E8F0;
	font-size: 44rpx;
	padding: 0 4rpx;
	transition: color 0.2s ease;
}

.star.active {
	color: #FFB400;
}

.score-text {
	color: #666;
	font-size: 26rpx;
	min-width: 60rpx;
	font-weight: 500;
}

.bottom-button {
	position: fixed;
	bottom: 0;
	left: 200rpx;
	right: 0;
	padding: 30rpx 40rpx;
	background: linear-gradient(180deg, rgba(255,255,255,0) 0%, #fff 20%);
}

.bottom-button button {
	background: linear-gradient(135deg, #4B9EF9 0%, #2F86F6 100%);
	color: #fff;
	border-radius: 45rpx;
	font-size: 32rpx;
	height: 90rpx;
	line-height: 90rpx;
	font-weight: 500;
	box-shadow: 0 6rpx 20rpx rgba(75,158,249,0.3);
	transition: all 0.3s ease;
}

.bottom-button button:active {
	transform: translateY(2rpx);
	box-shadow: 0 2rpx 10rpx rgba(75,158,249,0.2);
}
</style>
