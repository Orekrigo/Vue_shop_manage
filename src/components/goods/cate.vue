<template>
	<div>
		<!-- 面包屑导航区 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>商品管理</el-breadcrumb-item>
			<el-breadcrumb-item>商品分类</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图区 -->
		<el-card>
			<!-- 添加按钮区域 -->
			<el-row>
				<el-col>
					<el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
				</el-col>
			</el-row>

			<!-- 列表区域 渲染 -->
			<tree-table :columns="cateColumns" :data="cateList" :expand-type="false" :selection-type="false"
				:show-index="true" :show-row-hover="true" border class="tree-table" element-loading-text="拼命加载中"
				index-text="#">

				<!-- 第一个模板: 是否有效-->
				<template slot="isOk" slot-scope="scope">
					<i v-if="scope.row.cat_deleted === false" class="el-icon-success" style="color: lightgreen"></i>
					<i v-else class="el-icon-error" style="color: red"></i>
				</template>

				<!-- 第二个模板插槽 -->
				<template slot="order" slot-scope="scope">
					<el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
					<el-tag v-else-if="scope.row.cat_level === 1" size="mini" type="success">二级</el-tag>
					<el-tag v-else size="mini" type="warning">三级</el-tag>
				</template>

				<template slot="opt" slot-scope="scope">
					<el-button icon="el-icon-edit" size="mini" type="primary" @click="showEditDialog(scope.row.cat_id)">
						编辑
					</el-button>
					<el-button icon="el-icon-delete" size="mini" type="danger"
						@click="removeCateById(scope.row.cat_id)">删除
					</el-button>
				</template>

			</tree-table>

			<!-- 分页区域展示 -->
			<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
				:current-page="queryInfo.pagenum" :page-sizes="[10, 20, 30, 40]" :page-size="queryInfo.pagesize"
				layout="total, sizes, prev, pager, next, jumper" :total="total">
			</el-pagination>
		</el-card>

		<!-- 编辑的dialog对话框 -->
		<el-dialog title="编辑分类" :visible.sync="editCateDialogVisible" width="30%">
			<span>
				<!-- input框 -->
				<el-form ref="editCateFormRef" :model="editCateForm" label-width="100px">
					<el-form-item label="分类名称:">
						<el-input v-model="editCateForm.cat_name"></el-input>
					</el-form-item>
				</el-form>
			</span>
			<span slot="footer" class="dialog-footer">
				<el-button @click="editCateDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="editCate">确 定</el-button>
			</span>
		</el-dialog>

		<!-- 添加的dialog对话框 -->
		<el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="30%">
			<span>
				<el-form ref="addCateFormRef" :model="addCateForm" label-width="100px">
					<el-form-item label="分类名称: " prop="cat_name">
						<el-input v-model="addCateForm.cat_name"></el-input>
					</el-form-item>
					<el-form-item label="父级分类: ">
						<!-- options 用来指定数据源
			      cascadeProps 用来指定配置对象
			      -->
						<el-cascader v-model="selectedKeys" :change-on-select="true" :options="parentCateList"
							:props="cascadeProps" clearable @change="parentCateChanged"></el-cascader>
					</el-form-item>
				</el-form>


			</span>
			<span slot="footer" class="dialog-footer">
				<el-button @click="addCateDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="addCate">确 定</el-button>
			</span>
		</el-dialog>

	</div>
</template>

<script>
	export default {
		data() {
			return {
				queryInfo: {
					type: 3,
					pagenum: 1,
					pagesize: 5
				},
				cateList: [],
				total: 0,
				// tree-table 列表配置项
				cateColumns: [{
						label: '分类名称',
						prop: 'cat_name'
					},
					{
						label: '是否有效',
						// 表示, 将当前列定义为模板列
						type: 'template',
						// 表示当前列使用模板名称
						template: 'isOk'
					},
					{
						label: '排序',
						// 表示, 将当前列定义为模板列
						type: 'template',
						// 表示当前列使用模板名称
						template: 'order'
					},
					{
						label: '操作',
						// 表示, 将当前列定义为模板列
						type: 'template',
						// 表示当前列使用模板名称
						template: 'opt'
					}
				],
				editCateDialogVisible: false,
				editCateForm: {},
				addCateDialogVisible: false,
				parentCateList: [],
				addCateForm: {
					// 将要添加的分类的名称
					cat_name: '',
					// 父级分类的Id
				 cat_pid: 0,
					// 分类的等级, 默认要添加的是一级分类
					cat_level: 0
				},
				// 级联选择框 的下拉选中项 所以这是个数组类型
				selectedKeys: [],
				// 级联选择框的配置项
				cascadeProps: {
					value: 'cat_id',
					label: 'cat_name',
					children: 'children',
					// 从el-cascader标签的属性,中移动到这个cascadeProps中
					expandTrigger: 'hover'
				},
			}
		},
		created() {
			this.getCateList()
		},
		methods: {
			//页面列表数据
			async getCateList() {
				const {
					data: res
				} = await this.$http.get('categories', {
					params: this.queryInfo
				});
				if (res.meta.status !== 200) return this.$message.error('请求列表失败');
				//请求成功
				this.cateList = res.data.result;
				this.total = res.data.total;
			},
			async removeCateById(cat_id) {
				const {
					data: res
				} = await this.$http.delete(`categories/${cat_id}`);
				if (res.meta.status !== 200) return this.$message.error('删除失败');
				this.$message.success('删除成功');
				this.getCateList();
			},
			handleSizeChange(newsize) {
				this.queryInfo.pagesize = newsize;
				this.getCateList();
			},
			handleCurrentChange(newnum) {
				this.queryInfo.pagenum = newnum;
				this.getCateList();
			},
			async showEditDialog(cat_id) {
				// 通过接口  单条查询接口 把数据请求出来  然后赋值给editCateForm

				const {
					data: res
				} = await this.$http.get(`categories/${cat_id}`);
				if (res.meta.status !== 200) return this.$message.error('查询失败');
				//赋值操作
				this.editCateForm = res.data;
				this.editCateDialogVisible = true
			},
			async editCate() {
				const {
					data: res
				} = await this.$http.put(`categories/${this.editCateForm.cat_id}`, {
					cat_name: this.editCateForm.cat_name
				});
				if (res.meta.status !== 200) return this.$message.error('修改失败');
				this.$message.success('修改成功');
				this.editCateDialogVisible = false;
				this.getCateList();
			},
			showAddCateDialog() {
				this.getParentCateList()
				this.addCateDialogVisible = true;
			},
			//级联选择框的请求数据
			// 我为什么只查两级  没有第三级
			async getParentCateList() {
				const {
					data: res
				} = await this.$http.get('categories', {
					params: {
						type: 2
					}
				});
				this.parentCateList = res.data
			},
			parentCateChanged() {
				// 如果 selectedKeys数组中的length 大于 0, 证明选中的父级分类
				// 反之, 就说明没有选中任何父级分类
				console.log(this.selectedKeys);

				if (this.selectedKeys.length > 0) {
					// 父级分类的Id
					// 为什么要给addCateForm从新赋值 是因为  我要通过addCateform  当我添加接口里面的参数
					this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
					// 为当前分类的等级赋值
					this.addCateForm.cat_level = this.selectedKeys.length
				} else {
					// 父级分类的Id
					this.addCateForm.cat_pid = 0
					// 为当前分类的等级赋值
					this.addCateForm.cat_level = 0
				}
			},
			//添加操作
			async addCate() {
				const {
					data: res
				} = await this.$http.post('categories', this.addCateForm);
				if (res.meta.status !== 201) return this.$message.error('添加失败');
				this.$message.success('添加成功');
				this.getCateList();
				this.addCateDialogVisible = false;
			}
		}
	}
</script>

<style lang="less" scoped>
</style>
