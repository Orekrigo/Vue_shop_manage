<template>
	<div>
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>权限管理</el-breadcrumb-item>
			<el-breadcrumb-item>权限列表</el-breadcrumb-item>
		</el-breadcrumb>

		<el-card>
			<!-- 写我的table列表操作 stripe 隔行变色  border是边框 -->
			<el-table :data="rightList" border stripe>
				<el-table-column type="index" label="#"></el-table-column>
				<el-table-column label="权限名称" prop="authName"></el-table-column>
				<el-table-column label="路径" prop="path"></el-table-column>
				<el-table-column label="权限等级">
					<!-- 0  1  2 传参的 模板插槽 -->
					<template slot-scope="scope">
						<el-tag type="danger" v-if="scope.row.level==='0'">等级一</el-tag>
						<el-tag type="warning" v-if="scope.row.level==='1'">等级二</el-tag>
						<el-tag type="success" v-if="scope.row.level==='2'">等级三</el-tag>
					</template>
				</el-table-column>
			</el-table>
		</el-card>
	</div>
</template>

<script>
	// 第一步绘制页面
	// 写axios 像接口拿数据
	// 绑定到我的data层
	// table绑定数据源 并且执行渲染

	// 通过v-if 进行的判断 使用的依旧是模板插槽

	export default {
		data() {
			return {
				rightList: []
			}
		},
		created() {
			this.getRightList();
		},
		methods: {
			async getRightList() {
				const {
					data: res
				} = await this.$http.get('rights/list');
				if (res.meta.status !== 200) return this.$message.error('获取列表失败');
				console.log(res);
				//获取成功的情况
				// 要给我的data层进行一个赋值操作  这个data层这个变量是什么类型  一定是数组
				this.rightList = res.data;
			}
		}
	}
</script>

<style lang="less" scoped>
</style>
