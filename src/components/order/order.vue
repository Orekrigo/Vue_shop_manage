<template>
	<div>
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>订单管理</el-breadcrumb-item>
			<el-breadcrumb-item>订单列表</el-breadcrumb-item>
		</el-breadcrumb>
		<el-card>
			<!-- 搜索框区域 -->
			<el-row>
				<el-col :span="8">
					<el-input placeholder="请输入内容" >
						<el-button slot="append" icon="el-icon-search">></el-button>
					</el-input>
				</el-col>
			</el-row>
			<el-table :data="orderList" stripe border element-loading-text="拼命加载中" width="100%">
				<!-- 小项 -->
				<!-- 这个索引栏是数组下标 并非接口里面的id -->
				<el-table-column label="#" type="index"></el-table-column>
				<el-table-column label="订单编号" prop="order_number"></el-table-column>
				<el-table-column label="订单价格" prop="order_price"></el-table-column>
				<el-table-column label="是否付款" prop="pay_status">
					<!-- 模板插槽 -->
					<template slot-scope="scope">
						<el-tag v-if="scope.row.pay_status === '1'" type="success">已付款</el-tag>
						<el-tag v-else type="danger">未付款</el-tag>
					</template>
				</el-table-column>
				<el-table-column label="是否发货" prop="is_send"></el-table-column>
				<el-table-column label="下单时间" prop="create_time">
					<template slot-scope="scope">
						<!-- {{数据项 | 过滤器的函数名}} -->
						{{ scope.row.create_time | dateFormat }}
					</template>
				</el-table-column>
				<el-table-column label="操作">
					<!-- 模板插槽进行传参 -->
					<template slot-scope="scope">
						<el-button icon="el-icon-edit" size="mini" type="primary" @click="showBox"></el-button>
						<el-button icon="el-icon-location" size="mini" type="success"
							@click="showProgressBox(scope.row)"></el-button>
					</template>
				</el-table-column>
			</el-table>
			<!-- 分页操作 -->
			<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
				:current-page="queryInfo.pagenum" :page-sizes="[10, 20, 30, 40]" :page-size="queryInfo.pagesize"
				layout="total, sizes, prev, pager, next, jumper" :total="total">
			</el-pagination>
		</el-card>
		<!-- 修改地址的 dialog对话框 -->
		<el-dialog title="修改地址" :visible.sync="addressVisible" width="30%">
			<span>
				<el-form :model="addressForm" label-width="100px">
					<el-form-item label="省市区/县" prop="address1">
						<el-cascader v-model="addressForm.address1" :options="cityData" :props="cityProps" clearable
							expand-trigger="hover"></el-cascader>
					</el-form-item>
					<el-form-item label="详细地址" prop="detailAdd">
						<el-input v-model="addressForm.detailAdd"></el-input>
					</el-form-item>
				</el-form>
			</span>
			<span slot="footer" class="dialog-footer">
				<el-button @click="addressVisible = false">取 消</el-button>
				<el-button type="primary" @click="addressVisible = false">确 定</el-button>
			</span>
		</el-dialog>

		<!-- dialog 对话框 -->
		<el-dialog title="查看物流信息" :visible.sync="progressVisible" width="45%">
			<span>
				<!-- 时间轴 element-ui 组件 -->
				<el-timeline :reverse="true">
					<el-timeline-item v-for="(activity, index) in progressInfo" :key="index" :timestamp="activity.time">
						{{ activity.context }}
					</el-timeline-item>
				</el-timeline>
			</span>
			<span slot="footer" class="dialog-footer">
				<el-button type="primary" @click="progressVisible = false">确 定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
	import cityData from '../../utils/citydata'
	export default {
		data() {
			return {
				queryInfo: {
					query: '',
					pagenum: 1,
					pagesize: 10
				},
				orderList: [],
				total: 0,
				addressVisible: false,
				addressForm: {
					address1: [],
					detailAdd: ''
				},
				// 把我省市区的数据 绑定到我的data层当中
				cityData: cityData,
				cityProps: {
					expandTrigger: 'hover'
				},
				progressVisible: false,
				progressInfo: []
			}
		},
		created() {
			this.getOrdersList()
		},
		methods: {
			async getOrdersList() {
				const {
					data: res
				} = await this.$http.get('orders', {
					params: this.queryInfo
				});
				if (res.meta.status !== 200) return this.$message.error('请求接口失败');
				//请求接口成功的情况  我需要把res里面的data  赋值给我的数据层
				this.orderList = res.data.goods;
				this.total = res.data.total;
			},
			showBox() {
				this.addressVisible = true;
			},

			// 查看物流接口
			async showProgressBox(row) {
				// console.log(row);
				// 模拟接口数据  做假的数据源
				this.progressInfo = [{
						time: '2018-05-10 09:39:00',
						ftime: '2018-05-10 09:39:00',
						context: '已签收,感谢使用顺丰,期待再次为您服务',
						location: ''
					},
					{
						time: '2018-05-10 08:23:00',
						ftime: '2018-05-10 08:23:00',
						context: '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
						location: ''
					},
					{
						time: '2018-05-10 07:32:00',
						ftime: '2018-05-10 07:32:00',
						context: '快件到达 [北京海淀育新小区营业点]',
						location: ''
					},
					{
						time: '2018-05-10 02:03:00',
						ftime: '2018-05-10 02:03:00',
						context: '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
						location: ''
					},
					{
						time: '2018-05-09 23:05:00',
						ftime: '2018-05-09 23:05:00',
						context: '快件到达 [北京顺义集散中心]',
						location: ''
					},
					{
						time: '2018-05-09 21:21:00',
						ftime: '2018-05-09 21:21:00',
						context: '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
						location: ''
					},
					{
						time: '2018-05-09 13:07:00',
						ftime: '2018-05-09 13:07:00',
						context: '顺丰速运 已收取快件',
						location: ''
					},
					{
						time: '2018-05-09 12:25:03',
						ftime: '2018-05-09 12:25:03',
						context: '卖家发货',
						location: ''
					},
					{
						time: '2018-05-09 12:22:24',
						ftime: '2018-05-09 12:22:24',
						context: '您的订单将由HLA（北京海淀区清河中街店）门店安排发货。',
						location: ''
					},
					{
						time: '2018-05-08 21:36:04',
						ftime: '2018-05-08 21:36:04',
						context: '商品已经下单',
						location: ''
					}
				];
				this.progressVisible = true;

			},
			handleSizeChange(newSize) {
				// 我newsize  就是我点击之后的这个最新值  然后我给queryinfo 里面pagesize从新赋值
				this.queryInfo.pagesize = newSize
				this.getOrdersList()
			},
			handleCurrentChange(newPage) {
				// 从新赋值 从新请求 完成分页
				this.queryInfo.pagenum = newPage
				this.getOrdersList()
			}
		}
	}
</script>

<style lang="less" scoped>

</style>
