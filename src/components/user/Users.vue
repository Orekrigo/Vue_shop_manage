<template>
	<div>
		<!-- 面包屑导航 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>用户管理</el-breadcrumb-item>
			<el-breadcrumb-item>用户列表</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图区域 
		我应该在哪写这个样式
		
		-->
		<el-card class="box-card">
			<!-- 是两个格子  中间隔开的距离 -->
			<el-row :gutter="20">
				<!-- 文本搜索框 和 添加按钮部分 -->
				<el-col :span="8">
					<el-input placeholder="请输入内容" v-model="queryInfo.query">
						<el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
					</el-input>
				</el-col>
				<el-col :span="4">
					<el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
				</el-col>
			</el-row>

			<!-- table表格的渲染 -->

			<!-- :data 就是我的数据源 -->
			<el-table :data="userList" border>
				<el-table-column label="#" type="index">
				</el-table-column>
				<el-table-column prop="username" label="姓名" width="180">
				</el-table-column>
				<el-table-column prop="email" label="邮箱" width="180">
				</el-table-column>
				<el-table-column prop="mobile" label="电话">
				</el-table-column>
				<el-table-column prop="role_name" label="角色">
				</el-table-column>
				<el-table-column prop="mg_state" label="状态">
					<!-- 模板插槽 -->
					<template slot-scope="scope">
						<!-- 模板插槽的意义 就是把当前这一条数据  传递过来 -->
						<!-- v-model 双向数据绑定 true和false的地方 -->
						<!-- 什么事件 ()是什么内容  是咱们模板插槽里面的当前那条数据 -->
						
						<el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
					</template>
				</el-table-column>
				<el-table-column label="操作">
					<!-- 这需不需要模板插槽  一定是需要的 -->
					<template slot-scope="scope">
						<!-- 修改 -->
						<el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
						<!-- 删除 -->
						<el-button type="danger" icon="el-icon-delete" size="mini" @click="UserDelete(scope.row.id)"></el-button>
						<!-- 分配权限 -->
						
						<el-tooltip class="item" effect="dark" content="分配权限" placement="top">
							<el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
						</el-tooltip>
					</template>
				</el-table-column>
			</el-table>
			<el-pagination
			  @size-change="handleSizeChange"
			  @current-change="handleCurrentChange"
			  :current-page="queryInfo.pagenum"
			  :page-sizes="[3, 5, 8, 10]"
			  :page-size="queryInfo.pagesize"
			  layout="total, sizes, prev, pager, next, jumper"
			  :total="total">
			</el-pagination>
		</el-card>
		
		<!-- 思路总结
		1、当遇到form表单的时候
		:model=addForm
		form表单当中的  prop
		
		-->
		<!-- 添加对话框板块  addDialogVisible 就两个选项 true和false  true代表显示 false代表隐藏 -->
		<el-dialog title="添加用户" :visible.sync="addDialogVisible" width="30%">
		  <span>
			  <!-- 表单选项 -->
			  <el-form :rules="addFormRules" :model="addForm" label-width="80px">
			    <el-form-item label="用户名" prop="username">
			      <el-input v-model="addForm.username"></el-input>
			    </el-form-item>
				<el-form-item label="密码" prop="password">
				  <el-input type="password" v-model="addForm.password"></el-input>
				</el-form-item>
				<el-form-item label="邮箱" prop="emial">
				  <el-input v-model="addForm.email"></el-input>
				</el-form-item>
				<el-form-item label="手机号" prop="mobile">
				  <el-input v-model="addForm.mobile"></el-input>
				</el-form-item>
			  </el-form>
		  </span>
		  <span slot="footer" class="dialog-footer">
		    <el-button @click="addDialogVisible = false">取 消</el-button>
		    <el-button type="primary" @click="addUser">确 定</el-button>
		  </span>
		</el-dialog>
		
		
		<!-- 修改操作的dialog对话框 -->
		<el-dialog title="修改用户" :visible.sync="editDialogVisible" width="30%">
		  <span>
			  <el-form :model="editForm" :rules="editFormRules" label-width="80px">
			    <el-form-item label="用户名" prop="username">
			      <el-input v-model="editForm.username" disabled></el-input>
			    </el-form-item>
				<el-form-item label="邮箱" prop="email">
				  <el-input v-model="editForm.email"></el-input>
				</el-form-item>
				<el-form-item label="手机号" prop="mobile">
				  <el-input v-model="editForm.mobile"></el-input>
				</el-form-item>
			  </el-form>
		  </span>
		  <span slot="footer" class="dialog-footer">
		    <el-button @click="editDialogVisible = false">取 消</el-button>
		    <el-button type="primary" @click="EditUserInfo">确 定</el-button>
		  </span>
		</el-dialog>
		
		
		<!-- 分配权限对话框 -->
		<el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="30%">
		  <span>
			  <p>当前的用户：{{userinfo.username}}</p>
			  <p>当前的角色：{{userinfo.role_name}}</p>
			  <p>分配新角色：
			  <!-- 为什么下拉框这  再我的select  需要一个v-model 呢  
			   是因为 有很多个选项  但是  v-model 是记录你最后选择了哪个
			   也就是说  我选择哪个  selectRoleId 就等于我:value="item.id"
			   -->
			  	<el-select v-model="selectRoleId" placeholder="请选择">
			  		<el-option v-for="item in roleList" :key="item.id" :label="item.roleName" :value="item.id">
			  		</el-option>
			  	</el-select>
			  
			  </p>
		  </span>
		  <span slot="footer" class="dialog-footer">
		    <el-button @click="setRightDialogVisible = false">取 消</el-button>
		    <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
		  </span>
		</el-dialog>
		
	</div>
</template>

<script>
	export default {
		data() {
			return {
				queryInfo: {
					query: '',
					pagenum: 1,
					pagesize: 2
				},
				//定义一个变量 去接收我接口返回回来的数据
				userList: [],
				total: 0,
				addDialogVisible:false,
				// 为什么要以花括号对象的方式定义
				addForm:{
					username: '',
					password: '',
					email: '',
					mobile: ''
				},
				addFormRules:{
					username: [{
							required: true,
							message: '请输入登录名称',
							trigger: 'blur'
						},
						{
							min: 3,
							max: 10,
							message: '长度在 3 到 5 个字符',
							trigger: 'blur'
						}
					],
					password: [{
							required: true,
							message: '请输入密码',
							trigger: 'blur'
						},
						{
							min: 6,
							max: 15,
							message: '长度在 6 到 15 个字符',
							trigger: 'blur'
						}
					],
					email: [{
						required: true,
						message: '请输入邮箱',
						trigger: 'blur'
					}],
					mobile: [{
						required: true,
						message: '请输入手机',
						trigger: 'blur'
					}]
				},
				editDialogVisible:false,
				editForm:{
					username: '',
					email: '',
					mobile: ''
				},
				//定义修改的验证规则
				editFormRules:{
					password: [{
							required: true,
							message: '请输入密码',
							trigger: 'blur'
						},
						{
							min: 6,
							max: 15,
							message: '长度在 6 到 15 个字符',
							trigger: 'blur'
						}
					],
					email: [{
						required: true,
						message: '请输入邮箱',
						trigger: 'blur'
					}],
					mobile: [{
						required: true,
						message: '请输入手机',
						trigger: 'blur'
					}]
				},
				setRightDialogVisible:false,
				//定义一个变量 去绑定我接口返回的角色列表信息
				roleList: [],
				userinfo:{},
				selectRoleId:''
			}
		},
		created() {
			this.getUserList();
		},
		methods: {
			//我再写 表格数据的时候  优先去先取数据 然后再渲染
			async getUserList() {
				//发送请求 像我的接口当中  取数据  在这就是规定  await  和 async二者就像处对象的关系
				const {data: res} = await this.$http.get('users', {params: this.queryInfo});
				this.userList = res.data.users;
				// total 这个变量  记录我当前这个接口里共有多少条数据
				this.total = res.data.total
			},
			// 改变用户状态的代码
			async userStateChanged(row)
			{
				// 当我需要  再地址当中拼接参数的时候  我需要  ${}
				// 我在这埋坑了 这是新手都会犯错的地方 问题再单引号
				// 重点  一般新手都犯错 不是单引号  是反引号  才能解析变量
				const {data:res} = await this.$http.put(`users/${row.id}/state/${row.mg_state}`);
				if(res.meta.status!==200) return this.$message.error('设置状态失败');
				this.$message.success('设置状态成功');
			},
			handleSizeChange(newSize)
			{
				this.queryInfo.pagesize = newSize;
				this.getUserList();
			},
			handleCurrentChange(newPage)
			{
				this.queryInfo.pagenum = newPage;
				this.getUserList();
			},
			async addUser()
			{
				//第一个参数 是请求路径  第二参数 是像服务区传递的参数
				const {data:res} = await this.$http.post('users',this.addForm);
				if(res.meta.status!==201) return this.$message.error('添加用户失败');
				this.$message.success('添加用户成功');
				// 必须要在这关闭对话框
				this.addDialogVisible = false;
				// 我需要从新请求一下列表数据  然后才能把新添加的数据 绑定到我的table列表里
				this.getUserList();
				//少了一个步骤  我得把我的addfrom 进行清空吧
				this.addForm.username = '';
				this.addForm.password = '';
				this.addForm.email = '';
				this.addForm.mobile = '';
			},
			async UserDelete(id)
			{
				const {data:res} = await this.$http.delete(`users/${id}`);
				if(res.meta.status!==200) return this.$message.error('删除失败');
				this.$message.success('删除成功');
				this.getUserList();
			},
			async showEditDialog(id){
				//通过我的id单条查询数据
				const {data:res} = await this.$http.get(`users/${id}`);
				this.editForm = res.data;
				this.editDialogVisible=true;
			},
			// 修改用户信息操作
			async EditUserInfo(){
				console.log(this.editForm);
				const {data:res} = await this.$http.put(`users/${this.editForm.id}`,{email:this.editForm.email,mobile:this.editForm.mobile});
				console.log(res);
				if(res.meta.status!==200) return this.$message.error('修改用户失败');
				this.$message.success('修改用户成功');
				this.editDialogVisible=false;
			},
			//分配权限
			async setRole(row){
				//查询我所有角色列表的信息
				const {data:res} = await this.$http.get('roles');
				// console.log(res);
				this.roleList = res.data;
				// 把当前传递过来的数据  绑定到我的userinfo上 
				this.userinfo = row;
				this.setRightDialogVisible = true;
			},
			async saveRoleInfo()
			{
				//特殊处理  如果我下拉框没有选择 那么应该给用户提示
				if (!this.selectRoleId) return this.$message.error('请选择分配的角色');
				//发送axios
				const {data:res} = await this.$http.put(`users/${this.userinfo.id}/role`,{rid:this.selectRoleId});
				if(res.meta.status!==200) return this.$message.error('修改角色失败');
				this.$message.success('修改角色成功');
				
				//要不要刷新一下列表页
				this.getUserList();
				
				//我需不需要把dialog对话框关闭
				this.setRightDialogVisible = false;
				
			}
		}
	}
</script>

<style>
</style>
