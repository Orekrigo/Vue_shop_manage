<template>
	<div class="login_container" id="app">
		<div class="login_box">
			<div class="avatar_box">
				<img src="../assets/logo.png" alt="">
			</div>
			<el-form label-width="0px" class="login_form" :rules="LoginFormRules" :model="LoginForm" ref="LoginFormRef">
				<el-form-item prop="username">
					<el-input v-model="LoginForm.username" prefix-icon="iconfont icon-user"></el-input>
				</el-form-item>
				<el-form-item prop="password">
					<el-input type="password" v-model="LoginForm.password" prefix-icon="iconfont icon-3702mima">
					</el-input>
				</el-form-item>
				<el-form-item class="btns">
					<!-- 按钮区域 -->
					<el-button type="primary" @click="login">登录</el-button>
					<el-button type="info" @click="resetLoginForm">重置</el-button>
				</el-form-item>
			</el-form>

		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				LoginForm: {
					username: '',
					password: ''
				},
				LoginFormRules: {
					username: [{
						required: true,
						message: '请输入用户名',
						trigger: 'blur'
					}],
					password: [{
						required: true,
						message: '请输入密码',
						trigger: 'blur'
					}]
				}
			}
		},
		methods: {
			resetLoginForm() {
				this.$refs.LoginFormRef.resetFields();
			},
			async login() {
				const {
					data: res
				} = await this.$http.post('login', this.LoginForm)
				console.log(res);
				if (res.meta.status !== 200) return this.$message.error('登录失败');

				this.$message.success('登录成功');
				window.sessionStorage.setItem('token', res.data.token);
				this.$router.push('/home');

			}
		}
	}
</script>
<style lang="less" scoped>
	.login_container {
		background-color: #2b4b6b;
		height: 100%;
	}

	.login_box {
		background-color: #FFFFFF;
		width: 450px;
		height: 300px;
		border-radius: 3px;

		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
	}

	.avatar_box {
		width: 130px;
		height: 130px;
		border: 1px solid #eee;
		border-radius: 50%;
		padding: 10px;
		box-shadow: 0 0 10px #eee;
		position: absolute;
		left: 50%;
		transform: translate(-50%, -50%);
		background-color: #FFFFFF;

		img {
			width: 100%;
			height: 100%;
			border-radius: 50%;
			background-color: #eee;
		}
	}

	.login_form {
		position: absolute;
		bottom: 0;
		width: 100%;
		padding: 0 20px;
		box-sizing: border-box;
	}

	.btns {
		display: flex;
		justify-content: flex-end;
	}
</style>
