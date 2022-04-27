<template>
	<div>
		<!-- 面包屑导航区域 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>权限管理</el-breadcrumb-item>
			<el-breadcrumb-item>角色列表</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图区域 -->
		<el-card>
			<!-- 按钮区域 -->
			<el-row>
				<el-col>
					<el-button type="primary" @click="addDialogVisible=true">添加角色</el-button>
				</el-col>
			</el-row>

			<!-- table表格区域 -->
			<el-table :data="rolelist" border stripe>
				<!-- 再elementui当中  我想要下拉板 -->
				<el-table-column type="expand">
					<template slot-scope="scope">
						<!-- {{scope.row}} -->
						<!-- 一行 -->
						<!-- 思路
						 每一个children  都针对的是我一个级别  一三三级
						 我使用模板插槽 把当前这条数据传递过来了
						 
						 -->

						<el-row v-for="(item1,i1) in scope.row.children" :key="item1.id"
							:class="['bdbottom',i1===0?'bdtop':'','vcenter']">
							<el-col :span="5">
								<!-- 第一级 -->
								<el-tag closable>{{item1.authName}}</el-tag>
								<i class="el-icon-caret-right"></i>
							</el-col>
							<el-col :span="19">

								<el-row v-for="(item2,i2) in item1.children" :key="item2.id"
									:class="[i2===0?'':'bdtop','vcenter']">
									<el-col :span="6">
										<!-- 第二级 -->
										<el-tag type="success" closable>{{item2.authName}}</el-tag>
										<i class="el-icon-caret-right"></i>
									</el-col>
									<el-col :span="18">
										<!-- 第三级 -->
										<el-tag type="warning" closable v-for="(item3,i3) in item2.children"
											:key="item3.id" @close="removeRightById(scope.row,item3.id)">
											{{item3.authName}}</el-tag>
									</el-col>
								</el-row>
							</el-col>
						</el-row>

					</template>

				</el-table-column>
				<el-table-column type="index" label="#"></el-table-column>
				<el-table-column label="角色名称" prop="roleName"></el-table-column>
				<el-table-column label="角色描述" prop="roleDesc"></el-table-column>
				<el-table-column label="操作">
					<!-- 当遇到按钮需要传参的情况下 必须要用 模板插槽 -->
					<template slot-scope="scope">
						<el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">
							编辑</el-button>
						<el-button type="danger" icon="el-icon-delete" size="mini" @click="RoleDelete(scope.row.id)">删除
						</el-button>
						<el-button type="warning" icon="el-icon-setting" size="mini"
							@click="showSetRightDialog(scope.row)">分配权限</el-button>
					</template>
				</el-table-column>
			</el-table>
		</el-card>

		<!-- 添加角色的dialog对话框 -->
		<el-dialog title="添加角色" :visible.sync="addDialogVisible" width="30%">
			<span>
				<!-- 添加表单区域 -->
				<el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="80px">
					<el-form-item label="角色名称">
						<el-input v-model="addForm.roleName" prop="roleName"></el-input>
					</el-form-item>
					<el-form-item label="角色描述">
						<el-input v-model="addForm.roleDesc" prop="roleDesc"></el-input>
					</el-form-item>
				</el-form>
			</span>
			<span slot="footer" class="dialog-footer">
				<el-button @click="addDialogVisible=false">取 消</el-button>
				<el-button type="primary" @click="addUser">确 定</el-button>
			</span>
		</el-dialog>


		<!-- 定义修改的dialog对话框 -->
		<el-dialog title="修改角色" :visible.sync="editDialogVisible" width="30%">
			<span>
				<!-- 添加表单区域 -->
			 <el-form :model="editForm" label-width="80px">
					<el-form-item label="角色名称">
						<el-input v-model="editForm.roleName"></el-input>
					</el-form-item>
					<el-form-item label="角色描述">
						<el-input v-model="editForm.roleDesc"></el-input>
					</el-form-item>
				</el-form>
			</span>
			<span slot="footer" class="dialog-footer">
				<el-button @click="editDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="EditRolesInfo">确 定</el-button>
			</span>
		</el-dialog>

		<!-- 定义我的分配权限dialog对话框 -->
		<el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" :before-close="handleClose">
			<span>
				<el-tree :data="rightsList" show-checkbox node-key="id" default-expand-all
					:default-checked-keys="defKeys" ref="treeRef" :props="treePorps">
					<!-- :props 是el-treee 需要读取的配置项 -->
				</el-tree>
			</span>
			<span slot="footer" class="dialog-footer">
				<el-button @click="setRightDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="allotRights">确 定</el-button>
			</span>
		</el-dialog>


	</div>



</template>

<script>
	export default {
		data() {
			return {
				rolelist: [],
				addDialogVisible: false,
				addForm: {
					roleName: '',
					roleDesc: ''
				},
				addFormRules: {
					roleName: [{
							required: true,
							message: '请输入角色名称',
							trigger: 'blur'
						},
						{
							min: 2,
							max: 10,
							message: '长度在 2 到 5 个字符',
							trigger: 'blur'
						}
					],
					roleDesc: [{
							required: true,
							message: '请输入角色描述',
							trigger: 'blur'
						},
						{
							min: 1,
							max: 15,
							message: '长度在 1 到 15 个字符',
							trigger: 'blur'
						}
					],
				},
				editDialogVisible: false,
				editForm: {},
				setRightDialogVisible: false,
				//权限列表tree的数据
				rightsList: [],
				treePorps: {
					//规定好的  label就相当于咱们的文字部分
					label: 'authName',
					//你的子集是哪一个字段  咱们的子级是哪个数据项
					children: 'children'
				},
				defKeys: [],
				roleid: ''
			}
		},
		created() {
			this.getRoleList();
		},
		methods: {
			async getRoleList() {
				const {
					data: res
				} = await this.$http.get('roles');
				if (res.meta.status !== 200) return this.$message.error('获取列表数据失败');
				//获取成功 我需要把res.data 数据 赋值给我的data层
				this.rolelist = res.data;
			},
			async addUser() {
				const {
					data: res
				} = await this.$http.post('roles', this.addForm);
				if (res.meta.status !== 201) return this.$message.error('添加角色失败');
				this.$message.success('添加角色成功');
				this.getRoleList();
				this.addDialogVisible = false;
			},
			async RoleDelete(id) {
				const {
					data: res
				} = await this.$http.delete(`roles/${id}`);
				if (res.meta.status !== 200) return this.$message.error('删除失败');
				//删除成功
				this.$message.success('删除角色成功');
				//刷新我的列表
				this.getRoleList();
			},
			async showEditDialog(id) {

				const {
					data: res
				} = await this.$http.get(`roles/${id}`);
				if (res.meta.status !== 200) return this.$message.error('获取失败');
				console.log(res);
				//获取成功要给我的data数据层 进行一个赋值操作
				this.editForm = res.data;
				this.editDialogVisible = true;
			},
			async EditRolesInfo() {
				const {
					data: res
				} = await this.$http.put(`roles/${this.editForm.roleId}`, {
					roleName: this.editForm.roleName,
					roleDesc: this.editForm.roleDesc
				});
				if (res.meta.status !== 200) return this.$message.error('更改失败');
				this.$message.success('更改成功');
				this.getRoleList();
				//把dialog对话框关闭掉
				this.editDialogVisible = false
			},
			async removeRightById(row, rightid) {
				// row 代表当前这一行的数据
				// rightid 代表我权限id
				// `` 当路径当中出现了  变量的时候  需要用反引号解析
				const {
					data: res
				} = await this.$http.delete(`roles/${row.id}/rights/${rightid}`);
				if (res.meta.status !== 200) return this.$message.error('删除权限失败');
				this.$message.success('删除权限成功');
				//刷新列表进行数据请求
				// this.getRoleList();
				// 是因为  数据是双向数据绑定的

				row.children = res.data;
			},
			async showSetRightDialog(row) {
				this.roleid = row.id;
				const {
					data: res
				} = await this.$http.get('rights/tree');
				// console.log(res);
				// 把我从服务器接口请求回来的数据  绑定到我的data层当中 作为数据源进行使用
				if (res.meta.status !== 200) return this.$message.error('获取失败');
				this.rightsList = res.data;
				this.getLeafKeys(row, this.defKeys);
				this.setRightDialogVisible = true;
			},
			//封装  方便以后继续使用
			//通过递归的方式进行defKeys的赋值操作 是只允许有第三级权限
			getLeafKeys(node, arr) {
				// 如果没有children属性  那么他肯定是第三级选项
				if (!node.children) {
					return arr.push(node.id);
				}
				//如果是第一级  和第二级的选项  那么他自己调用自身  往下继续找
				node.children.forEach(item => this.getLeafKeys(item, arr));
			},
			async allotRights() {
				// [1,2,3]  把数组分割成字符换 "1,2,3"
				// 用... 进行省略  意思在这有可能会出现非常多的元素
				const keys = [
					...this.$refs.treeRef.getCheckedKeys(),
					...this.$refs.treeRef.getHalfCheckedKeys()
				]
				//console.log(this.$refs.treeRef.getCheckedKeys());
				//console.log(this.$refs.treeRef.getHalfCheckedKeys());
				//转化成字符串格式
				const idStr = keys.join(',');

				const {
					data: res
				} = await this.$http.post(`roles/${this.roleid}/rights`, {
					rids: idStr
				});

				if (res.meta.status !== 200) return this.$message.error('更新失败');
				this.$message.success('更新权限成功');
				this.getRoleList();
				this.setRightDialogVisible = false;
			},
			handleClose() {
				//函数相当于  在我dialog对话框  关闭之前所调用的函数
				this.defKeys = [];
			}

		}
	}
</script>

<style lang="less" scoped>
	.el-tag {
		margin: 7px;
	}

	.bdtop {
		border-top: 1px solid #eee;
	}

	.bdbottom {
		border-bottom: 1px solid #eee;
	}

	.vcenter {
		display: flex;
		align-items: center;
	}
</style>
