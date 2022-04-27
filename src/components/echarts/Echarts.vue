<template>
	<div>
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>数据统计</el-breadcrumb-item>
			<el-breadcrumb-item>数据报表</el-breadcrumb-item>
		</el-breadcrumb>
		<el-card class="echartscard">
			<div id="myChart" :style="{width: '100%', height: '550px'}"></div>
		</el-card>
	</div>
</template>

<script>
	let echarts = require('echarts/lib/echarts')
	require('echarts/lib/chart/bar')
	require('echarts/lib/component/tooltip')
	require('echarts/lib/component/title')
	export default {
		// 生命周期  需要用  mounted 挂载后的生命周期
		mounted() {
			this.drawLine();
		},
		methods: {
			async drawLine() {
				const {
					data: res
				} = await this.$http.get('reports/type/1');
				let myChart = echarts.init(document.getElementById('myChart'))
				myChart.setOption(res.data);
			}
		},
	}
</script>

<style lang="less" scoped>
	.echartscard{
		height: 100%;
	}
</style>
