<template>

    <section>

        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :model="filters" :inline="true">
                <el-form-item>
                    <el-input placeholder="关键字" v-model="filters.keyword"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getCourseTypes">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="warning" @click="add">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="courseTypes" v-loading="listLoading" highlight-current-row style="width: 100%;"
                  @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <!--索引-->
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="名称" sortable>
            </el-table-column>
            <el-table-column prop="logo" label="图标" sortable>
            </el-table-column>
            <el-table-column prop="path" label="路径" sortable>
            </el-table-column>
            <el-table-column prop="totalCount" label="总计" sortable>
            </el-table-column>
            <el-table-column prop="createTime" label="创建时间" sortable>
            </el-table-column>
            <el-table-column prop="updateTime" label="更新时间" sortable>
            </el-table-column>
            <el-table-column prop="parent.name" label="父类型">
            </el-table-column>
            <el-table-column prop="description" label="描述" sortable>
            </el-table-column>
            <!--            <el-table-column prop="states" label="是否启用" :formatter="formatStates" sortable>-->
            <!--            </el-table-column>-->

            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" type="warning" @click="edit(scope.row)">编辑</el-button>
                    <el-button size="small" type="danger" @click="del(scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" @click="delMore()">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="10"
                           :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!-- 添加/编辑对话框-->
        <el-dialog title="添加/修改" :visible.sync="formVisible" :close-on-click-modal="false">
            <el-form :model="courseType" label-width="80px" :rules="formRules" ref="courseType">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="courseType.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="图标" prop="logo">
                    <el-input v-model="courseType.logo" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="路径" prop="path">
                    <el-input v-model="courseType.path" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="总计" prop="totalCount">
                    <el-input v-model="courseType.totalCount" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述" prop="description">
                    <el-input v-model="courseType.description" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="父类型" prop="parent.name">
<!--                      <el-input v-model="courseType.email" auto-complete="off"></el-input>-->
                  <!--  <el-select v-model="parent.name" multiple placeholder="请选择" value-key="id">
                        <el-option
                                v-for="item in name"
                                :key="item.id"
                                :label="item.name"
                                :value="item">
                        </el-option>
                    </el-select>-->
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="formVisible = false">取消</el-button>
                <el-button type="primary" @click="save" :loading="saveLoading">提交</el-button>
            </div>
        </el-dialog>

    </section>

</template>

<script>

    export default {
        data() {
            /*var validatePass2 = (rule, value, callback) => {
                console.log(value); //确认密码
                if (value === '') {
                    callback(new Error('请再次输入密码'))
                } else if (value !== this.courseType.password) {
                    callback(new Error('两次输入密码不一致!'))
                } else if ((value.length < 6)) {
                    callback(new Error('输入密码太过简单!'))
                } else {
                    callback()
                }
            };*/
            /*var validPhone = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('请输入手机号'))
                } else {
                    const reg = /^1[3|4|5|7|8][0-9]\d{8}$/;
                    if (reg.test(value)) {
                        callback();
                    } else {
                        return callback(new Error('请输入正确的手机号!'));
                    }
                }
            };*/
            return {
                //对话框默认不显示,只有点击添加或修改的时候显示
                formVisible: false,
                listLoading: false,
                saveLoading: false,
                fileList: [],
                ids: [],
                filters: { //查询对象
                    keyword: ''
                },
                page: 1,//当前页,要传递到后台的
                total: 0, //分页总数
                courseTypes: [],
                courseType: {
                    id: null,
                    logo: '',
                    name: '',
                    path: '',
                    totalCount: '',
                    description: ''
                },
                formRules: {
                    name: [
                        {required: true, message: '请输入名称!', trigger: 'blur'}
                    ],
                    logo: [
                        {required: true, message: '请选择图标!', trigger: 'blur'}
                    ],
                    path: [
                        {required: true, message: '请输入路径!', trigger: 'blur'}
                    ],
                    totalCount: [
                        {required: true, message: '请输入总计!', trigger: 'blur'}
                    ],
                    description: [
                        {required: true, message: '请输入描述!', trigger: 'blur'}
                    ],
                    // repassword: [
                    //     {required: true, validator: validatePass2, trigger: 'blur'}
                    //
                    // ]
                }
            }
        },
        methods: {
            getCourseTypes() {
                //axios 发送Ajax请求后台获取数据
                let para = {//添加分页条件及高级查询条件
                    "page": this.page,
                    "keyword": this.filters.keyword,
                    //"rec": 0
                };
                //显示加载圈
                this.listLoading = true;
                this.$http.post("/course/courseType/", para)
                    .then(result => {
                        console.log(result.data.rows);
                        this.total = result.data.total;
                        this.courseTypes = result.data.rows;
                        //关闭加载圈
                        this.listLoading = false;
                    })
            },
            del(row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                        this.listLoading = true;
                        var id = row.id;
                        this.$http.delete("/course/courseType/" + id)
                            .then(result => {
                                this.listLoading = false;
                                //提示删除
                                if (result.data.success) {
                                    this.$message({
                                        message: '删除成功!',
                                        type: 'success'
                                    });
                                } else {
                                    this.$message({
                                        message: '删除失败!原因:' + result.data.message,
                                        type: 'error'
                                    });
                                }
                                //刷新页面
                                this.getCourseTypes();
                            })
                    }
                ).catch(() => {
                })
            },
            handleSelectionChange(val) {
                var id = []
                for (var i = 0; i < val.length; i++) {
                    id.push(val[i].id)
                }
                this.ids = id;
            },
            //批量删除
            delMore() {
                this.$confirm('确认删除多条记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                        this.listLoading = true;
                        var id = this.ids;
                        for (var i = 0; i < id.length; i++) {
                            this.$http.delete("/course/courseType/" + id[i])
                                .then(result => {
                                    this.listLoading = false;
                                    //提示删除
                                    if (result.data.success) {
                                        this.$message({
                                            message: '批量删除成功!',
                                            type: 'success'
                                        });
                                    } else {
                                        this.$message({
                                            message: '批量删除失败!原因:' + result.data.message,
                                            type: 'error'
                                        });
                                    }
                                });
                            //刷新页面
                            this.getCourseTypes();
                        }
                    }
                ).catch(() => {
                })
            },
            add() {
                //清空数据
                this.courseType = {
                    id: null,
                    logo: '',
                    name: '',
                    path: '',
                    totalCount: '',
                    description: ''
                };
                //打开对话框
                this.formVisible = true;
            },
            handleSuccess(response, file, fileList) {
                console.log(response);
                console.log(file);
                console.log(fileList);
                this.courseType.logo = response;
            },
            handleRemove(file, fileList) {
                console.log(file, fileList);
            },
            handlePreview(file) {
                console.log(file);
            },
            edit(row) {
                //拷贝后面对象的值到新对象,防止后面代码改动引起模型变化
                let courseTypeTemp = Object.assign({}, row);

                //拿到性别和状态和权限
               /*if (courseTypeTemp.sex === true) {
                    courseTypeTemp.sex = "true"
                } else if (courseTypeTemp.sex === false) {
                    courseTypeTemp.sex = "false"
                }

                if (courseTypeTemp.states === true) {
                    courseTypeTemp.states = "true"
                } else if (courseTypeTemp.states === false) {
                    courseTypeTemp.states = "false"
                }*/

                //回显数据
                this.courseType = courseTypeTemp;
                //显示
                this.formVisible = true;
            },
            save() {//合并了添加和修改
                this.$refs.courseType.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.saveLoading = true;
                            //拷贝后面对象的值到新对象,防止后面代码改动引起模型变化
                            let para = Object.assign({}, this.courseType);
                            console.log(para);
                            console.log("1111111111");
                            this.$http.put("/course/courseType/", para).then((res) => {
                                this.$message({
                                    message: '操作成功!',
                                    type: 'success'
                                });
                                //关闭刷新框
                                this.saveLoading = false;
                                //重置表单
                                this.$refs['courseType'].resetFields();
                                //关闭对话框
                                this.formVisible = false;
                                //刷新数据
                                this.getCourseTypes();
                            })
                        });
                    }
                })
            },
            handleCurrentChange(curentPage) {
                this.page = curentPage;
                this.getCourseTypes();
            },
            //初始化是否启用
            // formatStates(row) {
            //     return row.states === true ? '已启用' : row.states === false ? '未启用' : '未知状态';
            // },

        },
        mounted() {
            this.getCourseTypes()
        }
    }

</script>

<style scoped>

</style>