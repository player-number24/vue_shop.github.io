<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区域 -->
        <el-card>
            <!-- 警告区域 -->
            <el-alert
            show-icon
            title="警告提示的文案"
            type="warning"
            :closable="false">
          </el-alert>
          <!-- 选择商品分类 -->
          <el-row class="cat_opt">
              <el-col>
                  <span class="type">选择商品分类:</span>
                    <!-- 选择商品分类的级联选择器 -->
                    <el-cascader
                    v-model="selectedCateKeys"
                    :options="catelist"
                    :props="cateProps"
                    @change="handleChange"></el-cascader>
              </el-col>
          </el-row>
          <!-- tab区域 -->
          <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <el-tab-pane label="动态参数" name="many">
                <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
                <!-- 动态参数表格 --> 
                <el-table :data="manyTableData" border stripe>
                    <!-- 展开行 -->
                    <el-table-column type="expand">
                        <template slot-scope="scope">
                            <!-- 渲染tag标签 -->
                            <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close='handleClose(i,scope.row)'>{{item}}</el-tag>
                            <!-- 输入的文本框 -->
                            <el-input
                            class="input-new-tag"
                            v-if="scope.row.inputVisible"
                            v-model="scope.row.inputValue"
                            ref="saveTagInput"
                            size="small"
                            @keyup.enter.native="handleInputConfirm(scope.row)"
                            @blur="handleInputConfirm(scope.row)"
                            >
                            </el-input>
                            <!-- 添加按钮 -->
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
            <el-tab-pane label="静态属性" name="only">
                <el-button type="primary" size="mini"   :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
                <el-table :data="onlyTableData" border stripe>
                     <!-- 展开行 -->
                    <el-table-column type="expand">
                        <template slot-scope="scope">
                            <!-- 渲染tag标签 -->
                            <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close='handleClose(i,scope.row)'>{{item}}</el-tag>
                            <!-- 输入的文本框 -->
                            <el-input
                            class="input-new-tag"
                            v-if="scope.row.inputVisible"
                            v-model="scope.row.inputValue"
                            ref="saveTagInput"
                            size="small"
                            @keyup.enter.native="handleInputConfirm(scope.row)"
                            @blur="handleInputConfirm(scope.row)"
                            >
                            </el-input>
                            <!-- 添加按钮 -->
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="添加属性" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
          </el-tabs>
        </el-card>
        <!-- 信息提示框 -->
        <el-dialog
        :title="'添加'+titleText"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed">
        <!-- 添加参数的对话框 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
        </el-dialog>

        <!-- 修改参数对话框 -->
        <el-dialog
        title="编辑"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed">
        <!-- 添加参数的对话框 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="editForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
        </el-dialog>
    </div>
</template>

<script>
    export default{
        data () {
            return {
                // 定义商品列表为空
                catelist:[],
                cateProps:{
                    expandTrigger: 'hover',
                    value: 'cat_id',
                    label: 'cat_name',
                    children:'children'
                },
                // 选中的商品数组
                selectedCateKeys:[],
                // 被激活的页签的名称
                activeName:"many",
                // 动态参数的数据
                manyTableData:[],
                // 静态属性的数据
                onlyTableData:[],
                // 控制对话框的显示与隐藏
                addDialogVisible:false,
                // 添加参数的表单数据
                addForm:{
                   attr_name:"" 
                },
                // 添加表单验证规则对象
                addFormRules:{
                    attr_name:[{required: true,message:"请输入参数名称",trigger:'blur'},]
                },
                // 控制修改对话框的显示与隐藏
                editDialogVisible:false,
                // 添加参数的表单数据
                editForm:{},
                // 添加表单验证规则对象
                editFormRules:{
                      attr_name:[{required: true,message:"请输入参数名称",trigger:'blur'},]
                },
                // 控制按钮与文本框的1切换显示
                // inputVisible: false,
                // // 文本框中输入的内容
                // inputValue:''
            }
        },
        created () {
            this.getCateList();
        },
        methods: {
            async getCateList() {
                const {data: res} = await this.$http.get('categories')
                if(res.meta.status !== 200) {
                    return this.$message.error("获取商品分类失败！")
                }
                this.catelist =res.data
                // console.log(this.catelist);
            },
            // 级联选择框选中项变化，会触发这个函数
            handleChange() {
                this.getParamsData()

            },
             // tab页签点击事件的处理函数    
             handleTabClick() {
                console.log(this.activeName);
                this.getParamsData()
            },
            // 获取参数列表数据
            async getParamsData() {
                if(this.selectedCateKeys.length !== 3) {
                    this.selectedCateKeys = []
                    this.manyTableData = []
                    this.onlyTableData = []
                    return
                }
                // 根据所选分类的id，和当前所处的2面板，获取对应参数
                const {data: res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel: this.activeName}
                })
                if(res.meta.status !== 200) {
                    return this.$message.error("获取参数列表失败！")
                }
                res.data.forEach(item => {
                    item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                    // 控制文本框的显示与隐藏
                    item.inputVisible = false
                    // 文本框中输入值
                    item.inputValue = ''
                });
                console.log(res.data);
                if(this.activeName === 'many') {
                    this.manyTableData = res.data
                }else {
                    this.onlyTableData = res.data
                }
            },
            // 监听添加对话框的关闭事件
            addDialogClosed() {
                this.$refs.addFormRef.resetFields()
            },
            // 点击按钮，添加参数
            addParams() {
                this.$refs.addFormRef.validate(async valid => {
                    if(!valid) return
                    const {data: res} = await this.$http.post(`categories/${this.cateId}/attributes`,{
                        attr_name:this.addForm.attr_name,
                        attr_sel:this.activeName
                    })

                    if(res.meta.status !== 201) {
                        return this.$message.error('添加参数失败！')
                    }
                    this.$message.success('添加参数成功！')
                    this.addDialogVisible = false
                    this.getParamsData()
                })
            },
            // 点击按钮，展示修改的对话框
            async showEditDialog(attr_id) {
                // 查询当前参数信息
                const {data: res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,{
                    params:{attr_sel:this.activeName}
                })
                if(res.meta.status !== 200) {
                    return this.$message.error('获取信息失败!')
                }
                this.editForm = res.data
                this.editDialogVisible = true

            },
            // 重置对话框
            editDialogClosed() {
                this.$refs.editFormRef.resetFields()
            },
            // 点击按钮修改参数信息
            editParams(){
                this.$refs.editFormRef.validate(async valid => {
                    if(!valid) return
                    const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{
                        attr_name:this.editForm.attr_name,
                        attr_sel:this.activeName
                    })
                    if(res.meta.status !== 200) {
                        return this.$message.error('修改参数失败！')
                    }
                    this.$message.success('修改参数成功！')
                    this.getParamsData()
                    this.editDialogVisible = false

                })
            },
            // 根据id删除对应的参数项
            async removeParams(attr_id) {
                    const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                    }).catch(err => err);

                    // 用户取消了删除的操作
                    if(confirmResult !== 'confirm') {
                        return this.$message.info('已经删除！')
                    }

                    // 删除的业务逻辑
                    const {data: res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)

                    if(res.meta.status !== 200) {
                        return this.$message.error("删除参数失败！")
                    }
                    
                    this.$message.success('删除参数成功！')
                    this.getParamsData()
      
            },
            // 文本框失去焦点，或者按下enter都会触发
            async handleInputConfirm(row) {
                if(row.inputValue.trim().length === 0) {
                    row.inputValue = '',
                    row.inputVisible = false
                    return
                }
                // 如果没有return，则证明输入的内容，需要做后续处理
                row.attr_vals.push(row.inputValue.trim())
                row.inputValue = ''
                row.inputVisible = false
                // 需要发起请求，保存这次操作
                this.saveAttrVals(row)
            },
            // 将对attr_vals的操作，保存到数据库
            async saveAttrVals(row) {
                // 需要发起请求，保存这次操作
                const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
                    attr_name:row.attr_name,
                    attr_sel:row.attr_sel,
                    attr_vals:row.attr_vals.join(' ')
                })
                if(res.meta.status !== 200) {
                    return this.$message.error("修改信息失败！")
                }
                this.$message.success("修改信息成功！")

            },
            // 点击按钮没显示文本框
            showInput(row) {
                row.inputVisible = true
                // 让文本框自动获取焦点
                // $nextTick方法的作用，就是当页面上的元素被重新渲染后，才会指定回调函数中的代码
                this.$nextTick(_ => {
                    this.$refs.saveTagInput.$refs.input.focus();
                    });
            },
            // 删除对应参数可选项
            handleClose(i,row) {
                row.attr_vals.splice(i,1) 
                this.saveAttrVals(row)

            }
            },
        computed: {
            // 如果按钮需要被禁用，返回true，否则返回false
            isBtnDisabled() {
                if(this.selectedCateKeys.length !== 3) {
                    return true
                }
                return false
            },
            // 当前选中的三级分类的ID
            cateId() {
                if(this.selectedCateKeys.length === 3) {
                    return this.selectedCateKeys[2]
                }
                return null
            },
            // 动态计算标题的文本
            titleText() {
                if(this.activeName === 'many') {
                    return "动态参数"
                }else {
                    return "静态属性"
                }
            }
        }
           

    }
</script>

<style lang="less" scoped>
    .cat_opt {
        margin: 15px 0;
    }
    .type {
        margin-right: 10px;
    }
    .el-tag {
        margin: 10px;
    }
    .input-new-tag {
        width: 120px;
    }
</style>