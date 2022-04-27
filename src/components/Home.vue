<template>
	<el-container class="home-container">
		<el-header>
			<div>
				<img src="../assets/logo.png" alt="">
				<span>电商后台管理系统</span>
			</div>
			<el-button type="info" @click="logout">退出</el-button>
		</el-header>
		<el-container>

			<el-aside :width="isCollapse?'64px':'200px'">
				<div class="toggle-button" @click="toggleCollapse">|||</div>
				<el-menu class="el-menu-vertical-demo" background-color="#333744" text-color="#fff"
					active-text-color="#409EFF" unique-opened :collapse="isCollapse" :collapse-transition="false"
					router :default-active="activepath">

					<el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
						<template slot="title">
							<i :class="iconsObj[item.id]"></i>
							<span>{{item.authName}}</span>
						</template>
						<el-menu-item :index="'/'+subitem.path" v-for="subitem in item.children" :key="subitem.id"
							@click="saveNavState('/'+subitem.path)">
							<template slot="title">
								<i class="el-icon-menu"></i>
								<span>{{subitem.authName}}</span>
							</template>
						</el-menu-item>
					</el-submenu>
				</el-menu>
			</el-aside>
			<el-main>
				<router-view></router-view>
			</el-main>
		</el-container>
	</el-container>
</template>

<script>
	export default {
		data() {
			return {
				menulist: [],
				iconsObj: {
					'125': 'iconfont icon-user',
					'103': 'iconfont icon-tijikongjian',
					'101': 'iconfont icon-shangpin',
					'102': 'iconfont icon-danju',
					'145': 'iconfont icon-baobiao'
				},
				isCollapse: false,
				activepath:''
			}
		},
		created() {
			this.getMenuList();
			this.activepath = window.sessionStorage.getItem('activepath');
		},
		methods: {
			logout() {
				window.sessionStorage.clear();
				this.$router.push('/login');
			},
			async getMenuList() {
				const {
					data: res
				} = await this.$http.get('menus');
				this.menulist = res.data;
			},
			toggleCollapse() {
				this.isCollapse = !this.isCollapse
			},
			saveNavState(activepath) {
				// 我将把path 路径存储到我的session当中
				// console.log(path);
				window.sessionStorage.setItem('activepath', activepath);
				this.activepath = activepath;

			}
		}
	}
</script>

<style lang="less" scoped="scoped">
	.home-container {
		height: 100%;
	}

	.el-header {
		background-color: #373d41;
		display: flex;
		justify-content: space-between;
		padding-left: 0;
		align-items: center;
		color: #fff;
		font-size: 20px;

		>div {
			display: flex;
			align-items: center;

			span {
				margin-left: 15px;
			}
		}

		img {
			width: 13%;
		}
	}

	.el-aside {
		background-color: #333744;

		.el-menu {
			border-right: none;
		}
	}

	.iconfont {
		margin-right: 10px;
	}

	.toggle-button {
		background-color: #4a5064;
		color: #fff;
		font-size: 10px;
		line-height: 24px;
		text-align: center;
		// 我让我的三个||| 彼此之间不要那么近 有一定距离 0.3em
		letter-spacing: 0.3em;
		// 变成小手的样式
		cursor: pointer;
	}
</style>
