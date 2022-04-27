<template>
	<div>
		<!-- 面包屑导航区 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>商品管理</el-breadcrumb-item>
			<el-breadcrumb-item>分类参数</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图区域 -->
		<el-card>
			<!-- 提示文字  警告框 -->
			<el-alert :closable="false" show-icon title="注意: 只允许为三级分类设置相关参数!" type="warning">
			</el-alert>

			<el-row class="cat_opt">
				<el-col>
					<span>选择商品分类: </span>
					<!-- 选择商品分类的级联选择框 -->
					<el-cascader v-model="selectedKeys" :options="cateList" :props="cateProps" clearable
						expand-trigger="hover" @change="handleChange"></el-cascader>
				</el-col>
			</el-row>

			<!-- tag标签页的切换 -->
			<el-tabs v-model="activeName" @tab-click="handleTabClick">
				<el-tab-pane label="动态参数" name="many">
					<!-- 按钮区域 -->
					<el-button :disabled="isBtnDisabled" size="mini" type="primary">添加参数
					</el-button>

					<!-- 列表区域 -->
					<el-table :data="manyTableData" border stripe>
						<el-table-column type="expand">
							<template slot-scope="scope">
								<el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable
									@close="handleClose(i,scope.row)">
									{{ item }}
								</el-tag>

								<el-input v-if="scope.row.inputVisible" ref="saveTagInput"
									v-model="scope.row.inputValue" class="input-new-tag" size="small"
									@blur="handleInputConfirm(scope.row)"
									@keyup.enter.native="handleInputConfirm(scope.row)">
								</el-input>
								<el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+
									New Tag
								</el-button>
							</template>
						</el-table-column>
						<!--索引列-->
						<el-table-column label="#" type="index"></el-table-column>
						<el-table-column label="参数名称" prop="attr_name"></el-table-column>
						<el-table-column label="操作">
							<template slot-scope="scope">
								<el-button icon="el-icon-edit" size="mini" type="primary"
									@click="showEditDialog(scope.row)">编辑
								</el-button>
								<el-button icon="el-icon-delete" size="mini" type="danger"
									@click="removeParams(scope.row.attr_id)">删除
								</el-button>
							</template>
						</el-table-column>
					</el-table>
				</el-tab-pane>
				<el-tab-pane label="静态属性" name="only">
					<!-- 按钮区域 -->
					<el-button :disabled="isBtnDisabled" size="mini" type="primary">添加参数
					</el-button>

					<!-- 列表区域 -->
					<el-table :data="onlyTableData" border stripe>
						<el-table-column type="expand">
							<template slot-scope="scope">
								<el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable
									@close="handleClose(i,scope.row)">
									{{ item }}
								</el-tag>

								<el-input v-if="scope.row.inputVisible" ref="saveTagInput"
									v-model="scope.row.inputValue" class="input-new-tag" size="small"
									@blur="handleInputConfirm(scope.row)"
									@keyup.enter.native="handleInputConfirm(scope.row)">
								</el-input>
								<el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+
									New Tag
								</el-button>
							</template>
						</el-table-column>
						<!--索引列-->
						<el-table-column label="#" type="index"></el-table-column>
						<el-table-column label="参数名称" prop="attr_name"></el-table-column>
						<el-table-column label="操作">
							<template slot-scope="scope">
								<el-button icon="el-icon-edit" size="mini" type="primary"
									@click="showEditDialog(scope.row)">编辑
								</el-button>
								<el-button icon="el-icon-delete" size="mini" type="danger"
									@click="removeParams(scope.row.attr_id)">删除
								</el-button>
							</template>
						</el-table-column>
					</el-table>
				</el-tab-pane>
			</el-tabs>
		</el-card>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				cateList: [],
				cateProps: {
					value: 'cat_id',
					label: 'cat_name',
					children: 'children',
					// 从el-cascader标签的属性,中移动到这个cascadeProps中
					expandTrigger: 'hover'
				},
				selectedKeys: [],
				activeName: 'many',
				manyTableData: [],
				onlyTableData: []
			}
		},
		// 计算属性进行设置的
		computed: {
			isBtnDisabled() {
				return this.selectedKeys.length !== 3
			},
			cateId() {
				// selectedKeys 我集连选择框的结果集  并且是一个数组类型 存储的是id
				// 取出我第三级的id值
				if (this.selectedKeys.length === 3) {
					return this.selectedKeys[2]
				}
				return null
			}
		},
		created() {
			// 只要页面当中一加载 我就需要请求接口
			this.getCategories();
		},
		methods: {
			// 获取级联选择框 数据的函数
			async getCategories() {
				const {
					data: res
				} = await this.$http.get('categories');
				if (res.meta.status !== 200) return this.$message.error('获取列表失败');
				//获取成功的情况 我需要把 data赋值给我的数据层
				this.cateList = res.data
			},
			//级联选择框事件
			handleChange() {
				this.getParamsData()
			},
			//tab切换栏事件
			handleTabClick() {
				this.getParamsData()
			},
			//封装的请求函数
			async getParamsData() {
				// 如果我的选项 不是第三级  那么我给我的所有的东西 都清空
				if (this.selectedKeys.length !== 3) {
					this.selectedKeys = []
					this.manyTableData = []
					this.onlyTableData = []
				} else {
					//我的选项是第三级的情况
					// 请求我的接口
					const {
						data: res
					} = await this.$http.get(`categories/${this.cateId}/attributes`, {
						params: {
							sel: this.activeName
						}
					});

					// 将字符串 转化成数组
					res.data.forEach(item => {
						// 三元表达式
						item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
						// 控制输入框与按钮的切换形式
						item.inputVisible = false
						// 文本框中输入的值
						item.inputValue = ''
					})

					if (this.activeName === 'many') {
						this.manyTableData = res.data
					} else {
						this.onlyTableData = res.data
					}
				}
			},
			//单独删除某一个tag标签
			handleClose(index, row) {
				// 我把attr_vals 里面这个数组  删除这一项
				row.attr_vals.splice(index, 1)
				// 然后请求接口
				this.saveAttrVals(row)
			},
			async saveAttrVals(row) {
				const {
					data: res
				} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
					attr_name: row.attr_name,
					attr_sel: this.activeName,
					// 特殊处理  因为是数据 所以转化为字符串 传递给接口当中
					attr_vals: row.attr_vals.join(' ')
				});
				if (res.meta.status !== 200) return this.$message.error('修改失败');
				this.$message.success('修改成功');

			},
			showInput(row) {
				row.inputVisible = true
				// 我需要自动定位input焦点
				this.$nextTick(_ => {
					this.$refs.saveTagInput.$refs.input.focus()
				})
			},
			//添加tag操作
			handleInputConfirm(row) {
				//防止用户  违规操作  提交空值
				if (row.inputValue.trim().length === 0) {
					row.inputVisible = false
					row.inputValue = ''
					return
				}
				// 如果没有return, 这里证明输入的内容, 需要做后序处理
				row.attr_vals.push(row.inputValue.trim())
				row.inputValue = ''
				row.inputVisible = false
				// 发起请求, 保存这次操作
				this.saveAttrVals(row)
			},
			async removeParams(attrId) {
				const {
					data: res
				} = await this.$http.delete(`categories/${this.cateId}/attributes/${attrId}`);
				if (res.meta.status !== 200) return this.$message.error('删除失败');
				this.$message.success('删除成功');
				this.getCategories();
			}
		}
	}
</script>

<style lang="less" scoped>
	.cat_opt {
		margin: 15px 0;
	}

	.el-tag+.el-tag {
		margin-left: 10px;
	}

	.button-new-tag {
		margin-left: 10px;
		height: 32px;
		line-height: 30px;
		padding-top: 0;
		padding-bottom: 0;
	}

	.input-new-tag {
		width: 90px;
		margin-left: 10px;
		vertical-align: bottom;
	}
</style>
