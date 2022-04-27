<template>
	<div>
		<!-- 面包屑导航 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>商品管理</el-breadcrumb-item>
			<el-breadcrumb-item>修改商品</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图区域 -->
		<el-card>
			<!-- 提示区 -->
			<el-alert :closable="false" center show-icon title="添加商品信息" type="info">
			</el-alert>
			<!-- 步骤条 当一个类似于 数字的字符串 想转变成数字  后面直接减零  可以直接转变 -->
			<el-steps :space="200" :active="activeIndex-0" align-center finish-status="success">
				<el-step title="基本信息"></el-step>
				<el-step title="商品参数"></el-step>
				<el-step title="商品属性"></el-step>
				<el-step title="商品图片"></el-step>
				<el-step title="商品内容"></el-step>
				<el-step title="完成"></el-step>
			</el-steps>

			<!-- tabs 切换栏 -->
			<el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-position="top" label-width="100px">
				<el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTabLeave"
					@tab-click="tabClicked">
					<el-tab-pane label="基本信息" name="0">
						<el-form-item label="商品名称" prop="goods_name">
							<el-input v-model="addForm.goods_name"></el-input>
						</el-form-item>
						<el-form-item label="商品价格" prop="goods_price">
							<el-input v-model="addForm.goods_price"></el-input>
						</el-form-item>
						<el-form-item label="商品重量" prop="goods_weight">
							<el-input v-model="addForm.goods_weight"></el-input>
						</el-form-item>
						<el-form-item label="商品数量" prop="goods_number" type="number">
							<el-input v-model="addForm.goods_number"></el-input>
						</el-form-item>
						<el-form-item label="商品分类" prop="goods_cat">
							<!-- 集联选择框 -->
							<el-row>
								<el-col>
									<!-- 选择商品分类的级联选择框 -->
									<el-cascader v-model="addForm.goods_cat" :options="cateList" :props="cateProps"
										clearable @change="handleChange">
									</el-cascader>
									<span>再次选择 会更改商品分类 如果不选择 那么默认当前分类</span>
								</el-col>
							</el-row>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品参数" name="1">
						<el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
							<el-checkbox-group v-model="item.attr_vals">
								<el-checkbox v-for="(cb,i)  in item.attr_vals" :key="i" :label="cb" border size="small">
								</el-checkbox>
							</el-checkbox-group>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品属性" name="2">
						<el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
							<el-input v-model="item.attr_vals"></el-input>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品图片" name="3">
						<el-upload :action="uploadURL" :on-remove="handleRemove" :headers="headerObj"
							:on-success="handleSuccess" list-type="picture">
							<el-button size="small" type="primary">点击上传</el-button>
							<div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
						</el-upload>
					</el-tab-pane>
					<el-tab-pane label="商品内容" name="4">
						<!-- 富文本编辑器的地方 -->
						<quill-editor v-model="addForm.goods_introduce">
						</quill-editor>
						<el-button class="btnAdd" type="primary" @click="edit">修改商品</el-button>
					</el-tab-pane>
				</el-tabs>
			</el-form>
		</el-card>
	</div>
</template>

<script>
	// 明日重点
	// 项目结束  git版本控制器的配置
	//  上传 拉取  9点
	import _ from 'lodash'
	export default {
		data() {
			return {
				activeIndex: '0',
				addForm: {
					goods_name: '',
					goods_price: 0,
					goods_weight: 0,
					goods_number: 0,
					goods_cat: [],
					pics: [],
					goods_introduce: '',
					attrs: []
				},
				addFormRules: {
					goods_name: [{
						required: true,
						message: '请输入商品名称',
						trigger: 'blur'
					}],
					goods_price: [{
						required: true,
						message: '请输入商品价格',
						trigger: 'blur'
					}],
					goods_weight: [{
						required: true,
						message: '请输入商品重量',
						trigger: 'blur'
					}],
					goods_number: [{
						required: true,
						message: '请输入商品数量',
						trigger: 'blur'
					}],
					goods_cat: [{
						required: true,
						message: '请输入商品分类',
						trigger: 'blur'
					}],
				},
				cateList: [],
				cateProps: {
					expandTrigger: 'hover',
					label: 'cat_name',
					value: 'cat_id',
					children: 'children'
				},
				manyTableData: [],
				onlyTableData: [],
				headerObj: {
					Authorization: window.sessionStorage.getItem('token')
				},
				uploadURL: 'http://vueshop.pixiv.download/api/private/v1/upload',
			}
		},
		created() {
			this.getData();
			this.getCateList();
		},
		computed: {
			cateId() {
				if (this.addForm.goods_cat.length === 3) {
					return this.addForm.goods_cat[2]
				}
				return null
			}
		},
		methods: {
			async getData() {
				const id = this.$route.query.goodsid;
				// console.log(id);
				const {
					data: res
				} = await this.$http.get(`goods/${id}`);
				// console.log(res);
				//  我把goods_cat 进行分割
				res.data.goods_cat = res.data.goods_cat.split(',');

				this.addForm = res.data;
			},
			async getCateList() {
				const {
					data: res
				} = await this.$http.get('categories');
				if (res.meta.status !== 200) return this.$message.error('获取数据失败');
				//获取成功
				this.cateList = res.data
			},
			//选择框 发生改变的时候 所触发的事件
			handleChange() {
				// 我选择的可能是第一级  或者第二级别
				if (this.addForm.goods_cat.length !== 3) {
					this.addForm.goods_cat = []
				}
			},
			beforeTabLeave(activename, oldactivename) {
				if (oldactivename === '0' && this.addForm.goods_cat.length !== 3) {
					this.$message.error('请先选择商品分类');
					return false;
				}
			},
			async tabClicked() {
				// 当我点击了 第二个板块  我去请求接口 返回我的数据
				// 第二个板块的接口请求
				if (this.activeIndex === '1') {
					const {
						data: res
					} = await this.$http.get(`categories/${this.cateId}/attributes`, {
						params: {
							sel: 'many'
						}
					});
					if (res.meta.status !== 200) return this.$message.error('请求接口失败');
					//赋值给我的data  方便我的遍历
					// console.log(res.data);
					// 要把我attr_vals 转化成数组类型 我页面才能进行正常的数据遍历
					res.data.forEach(item => {
						item.attr_vals = item.attr_vals.substr(1);
						item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(',')
					})
					this.manyTableData = res.data
				}
				if (this.activeIndex === '2') {
					const {
						data: res
					} = await this.$http.get(`categories/${this.cateId}/attributes`, {
						params: {
							sel: 'only'
						}
					});
					if (res.meta.status !== 200) return this.$message.error('请求接口失败');
					//赋值给我的data  方便我的遍历
					// console.log(res.data);
					// 要把我attr_vals 转化成数组类型 我页面才能进行正常的数据遍历

					this.onlyTableData = res.data
				}
			},
			handleRemove(file) {
				console.log(file);
				const filePath = file.response.data.tmp_path
				// 2.从pics数组中, 找到这个图片对应的索引值
				const index = this.addForm.pics.findIndex(item => {
					return item.pic === filePath
				})
				// 3.调用数组的splice方法, 把图片信息对象, 从pics数组中移除
				this.addForm.pics.splice(index, 1)
			},
			handleSuccess(response) {
				// 传递的参数  自带的参数  服务器反馈回来的结果集
				const picInfo = {
					pic: response.data.tmp_path
				}
				// 2.将图片信息对象, push到 pics数组中
				this.addForm.pics.push(picInfo)
			},
			// 添加商品的事件
			async edit() {
				// 叫表单预验证 当我输入的东西和我本身的验证规则 不符合的时候  我valid 自动为false
				this.$refs.addFormRef.validate(async valid => {
					// 当我的valid 为false的时候 就证明我有没写的表单
					if (!valid) {
						return this.$message.error('请填写必要的表单项!')
					}

					// 发起请求  请求添加商品接口 对象的深拷贝
					// 为什么要这么做
					// 是因为 v-model 双向数据绑定 我下方一旦把数组更改成字符串
					// 那么 我上面的功能就失效 不好使了
					const form = _.cloneDeep(this.addForm);
					form.goods_cat = form.goods_cat.join(',');
					// 处理动态参数
					this.manyTableData.forEach(item => {
						const newInfo = {
							attr_id: item.attr_id,
							attr_value: item.attr_vals.join(' ')
						}
						this.addForm.attrs.push(newInfo)
					})
					// 处理静态属性
					this.onlyTableData.forEach(item => {
						const newInfo = {
							attr_id: item.attr_id,
							attr_value: item.attr_vals
						}
						this.addForm.attrs.push(newInfo)
					})
					form.attrs = this.addForm.attrs;
					const {
						data: res
					} = await this.$http.put(`goods/${this.$route.query.goodsid}`, form);
					console.log(res)
					if (res.meta.status !== 200) return this.$message.error('添加商品失败');
					//添加商品成功
					this.$message.success('修改商品成功');
					// 编程导航
					this.$router.push('/goods')
				})
			}
		}
	}
</script>

<style lang="less" scoped>
</style>
