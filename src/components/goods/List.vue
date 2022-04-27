<template>
	<div>
		<!-- 面包屑导航 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>商品管理</el-breadcrumb-item>
			<el-breadcrumb-item>商品列表</el-breadcrumb-item>
		</el-breadcrumb>

		<el-card>
			<el-row :gutter="20">
				<el-col :span="8">
					<!-- 搜索与添加区域 -->
					<el-input v-model="queryInfo.query" @clear="getGoodsList" @keyup.enter.native="getGoodsList"
						class="input-with-select" clearable placeholder="请输入内容">
						<el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
					</el-input>
				</el-col>
				<el-col :span="4">
					<!-- 和以往咱们用dialog  不同 这是直接跳转到其他组件当中了 -->
					<el-button type="primary" @click="goAddpage">添加商品</el-button>
				</el-col>
			</el-row>

			<!-- table列表区域 先获取数据  然后进行渲染 -->
			<el-table :data="goodsList" :stripe="true" border element-loading-text="拼命加载中" width="100%">
				<el-table-column label="#" type="index">
				</el-table-column>
				<el-table-column label="商品名称" prop="goods_name">
				</el-table-column>
				<el-table-column label="商品价格(元)" prop="goods_price" width="95px">
				</el-table-column>
				<el-table-column label="数量" prop="goods_number" width="70px">
				</el-table-column>
				<el-table-column label="创建时间" prop="add_time" width="140px">
					<template slot-scope="scope">
						{{ scope.row.add_time | dateFormat }}
					</template>
				</el-table-column>
				<el-table-column label="状态" prop="goods_state" width="95px">
					<template slot-scope="scope">
						<el-tag v-if="scope.row.goods_state === 0" type="info">未通过</el-tag>
						<el-tag v-else-if="scope.row.goods_state === 1" type="warning">审核中</el-tag>
						<el-tag v-else type="success">已审核</el-tag>
					</template>
				</el-table-column>
				<el-table-column label="操作" width="130px">
					<template slot-scope="scope">
						<!-- 修改按钮 -->
						<el-tooltip :enterable="false" class="item" content="修改商品" effect="dark" placement="top">
							<el-button icon="el-icon-edit" size="mini" type="primary"
								@click="editGoodsById(scope.row.goods_id)"></el-button>
						</el-tooltip>

						<!-- 删除按钮-->
						<el-tooltip :enterable="false" class="item" content="删除商品" effect="dark" placement="top">
							<el-button icon="el-icon-delete" size="mini" type="danger"
								@click="removeGoodsById(scope.row.goods_id)"></el-button>
						</el-tooltip>

					</template>
				</el-table-column>
			</el-table>

			<!-- 分页板块 -->
			<el-pagination :current-page="queryInfo.pagenum" :page-size="queryInfo.pagesize" :page-sizes="[5,10,15,20]"
				:total="total" background layout="total, sizes, prev, pager, next, jumper"
				@size-change="handleSizeChange" @current-change="handleCurrentChange">
			</el-pagination>
		</el-card>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				queryInfo: {
					query: '',
					pagenum: 1,
					pagesize: 5
				},
				goodsList: [],
				total: 0
			}
		},
		created() {
			this.getGoodsList()
		},
		methods: {
			async getGoodsList() {
				const {
					data: res
				} = await this.$http.get('goods', {
					params: this.queryInfo
				});
				if (res.meta.status !== 200) return this.$message.error('请求接口失败');
				//请求接口成功 赋值操作
				this.goodsList = res.data.goods;
				this.total = res.data.total;
			},
			handleSizeChange(newsize) {
				this.queryInfo.pagesize = newsize;
				this.getGoodsList()
			},
			handleCurrentChange(newpage) {
				this.queryInfo.pagenum = newpage;
				this.getGoodsList()
			},
			async removeGoodsById(id) {
				const {
					data: res
				} = await this.$http.delete(`goods/${id}`);
				if (res.meta.status !== 200) return this.$message.error('删除失败');
				this.$message.success('删除成功');
				this.getGoodsList()
			},
			// 只有一点  跳转到我的add组件  也就是我的添加组件当中
			goAddpage() {
				// 编程式导航  一定要用 this.$router.push('即将跳转的路由地址')
				this.$router.push('/goods/add')
			},
			editGoodsById(id) {

				this.$router.push({
					path: '/goods/edit',
					query: {
						goodsid: id
					}
				});
			}
		}
	}
</script>

<style>
</style>
