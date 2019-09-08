<template>

    <section>

        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :model="filters" :inline="true">
                <el-form-item>
                    <el-input placeholder="关键字" v-model="filters.keyword"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getCourses">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="warning" @click="add">新增</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="add">课程营销</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="warning" @click="add">课程图片</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary">课程阶段</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="success" @click="online" :disabled="this.sels.length===0">上线</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="info" @click="offline" :disabled="this.sels.length===0">下线</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="courses" v-loading="listLoading" highlight-current-row style="width: 100%;"
                  @selection-change="selsChange">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <!--索引-->
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="名称">
            </el-table-column>
            <el-table-column prop="courseType.name" label="类型">
            </el-table-column>
            <el-table-column prop="tenantName" label="机构">
            </el-table-column>
            <el-table-column prop="userName" label="创建者">
            </el-table-column>
            <el-table-column prop="status" label="状态">
                <template slot-scope="scope">
                    <span v-if="scope.row.status!=null && scope.row.status==0" style="color: red">下线</span>
                    <span v-else style="color: green">上线</span>
                </template>
            </el-table-column>
            <el-table-column prop="startTime" label="上线时间">
            </el-table-column>
            <el-table-column prop="endTime" label="下线时间">
            </el-table-column>
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
            <el-form :model="course" label-width="80px" :rules="formRules" ref="course">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="course.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="适用人群" prop="users">
                    <el-input v-model="course.users" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="课程等级" prop="grade">
                    <el-radio-group v-model="course.grade">
                        <el-radio v-for="courseLevel in courseLevels"
                                  :label="courseLevel.id">{{courseLevel.name}}
                        </el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="课程类型" prop="courseTypeId">
                    <!--<el-input v-model="course.courseTypeId" auto-complete="off"></el-input>-->
                    <el-cascader v-model="course.courseTypeId"
                                 :options="typeTree"
                                 :props="{ checkStrictly: true,value:'id',label:'name'}"
                                 clearable></el-cascader>
                </el-form-item>
                <el-form-item label="简介" prop="intro">
                    <el-input v-model="course.detail.intro" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="详情" prop="description">
                    <el-input v-model="course.detail.description" auto-complete="off"></el-input>
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
            return {
                //对话框默认不显示,只有点击添加或修改的时候显示
                formVisible: false,
                listLoading: false,
                saveLoading: false,
                fileList: [],
                filters: { //查询对象
                    keyword: ''
                },
                page: 1,//当前页,要传递到后台的
                total: 0, //分页总数
                courses: [],
                courseLevels: [],
                course: {
                    id: null,
                    name: '',
                    users: '',
                    courseTypeId: null,
                    grade: null,
                    detail: {
                        intro: '',
                        description: ''
                    }
                },
                formRules: {},
                sels: [],
                delids: [],
                typeTree: []
            }
        },
        methods: {
            getCourses() {
                //axios 发送Ajax请求后台获取数据
                let para = {//添加分页条件及高级查询条件
                    "page": this.page,
                    "keyword": this.filters.keyword,
                    //"rec": 0
                };
                //显示加载圈
                this.listLoading = true;
                this.$http.post("/course/course/", para)
                    .then(result => {
                        console.log(result.data.rows);
                        this.total = result.data.total;
                        this.courses = result.data.rows;
                        //关闭加载圈
                        this.listLoading = false;
                    })
            },
            getTypeTree() {
                //分页查询
                this.$http.get("/course/courseType/treeData")
                    .then(result => {
                        this.typeTree = result.data;
                    });
            },
            online() {
                console.log(this.sels) //是对象数组
                //map会做遍历,每遍历一次是一个元素,把所有id组装一个数组
                var ids = this.sels.map(item => item.id);  //[1,2,3]
                this.$confirm('确认上线选中的课程吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.post("/course/course/onLine", ids).then((result) => {
                        // AjaxResult转换结果
                        if (result.data.success) {
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                        } else {
                            this.$message({
                                message: result.data.message,
                                type: 'error'
                            });
                        }
                        //刷新数据
                        this.getCourses();
                    });
                }).catch(() => {
                });
            },
            offline() {
                console.log(this.sels) //是对象数组
                //map会做遍历,每遍历一次是一个元素,把所有id组装一个数组
                var ids = this.sels.map(item => item.id);  //[1,2,3]
                this.$confirm('确认下线选中的课程吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.post("/course/course/offLine", ids).then((result) => {
                        // AjaxResult转换结果
                        if (result.data.success) {
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                        } else {
                            this.$message({
                                message: result.data.message,
                                type: 'error'
                            });
                        }
                        //刷新数据
                        this.getCourses();
                    });
                }).catch(() => {
                });
            },
            selsChange(sels) {
                var id = [];
                for (var i = 0; i < sels.length; i++) {
                    id.push(sels[i].id)
                }
                this.delids = id;
                this.sels = sels;
            },
            getCourseLevels() {
                //发送请求到后台获取数据
                this.$http.get("/sysmanage/systemdictionaryitem/sn?sn=courseLevel")
                    .then(result => {
                        this.courseLevels = result.data;
                    });

            },
            del(row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                        this.listLoading = true;
                        var id = row.id;
                        this.$http.delete("/course/course/" + id)
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
                                this.getCourses();
                            })
                    }
                ).catch(() => {
                })
            },
            //批量删除
            delMore() {
                this.$confirm('确认删除多条记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                        this.listLoading = true;
                        var id = this.delids;
                        for (var i = 0; i < id.length; i++) {
                            this.$http.delete("/course/course/" + id[i])
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
                            this.getCourses();
                        }
                    }
                ).catch(() => {
                })
            },
            add() {
                //清空数据
                this.course = {
                    id: null,
                    name: '',
                    users: '',
                    courseTypeId: null,
                    grade: null,
                    detail: {
                        intro: '',
                        description: ''
                    }
                };
                //打开对话框
                this.formVisible = true;
                this.getCourseLevels();
                this.getTypeTree();
            },
            handleSuccess(response, file, fileList) {
                console.log(response);
                console.log(file);
                console.log(fileList);
                this.course.logo = response;
            },
            handleRemove(file, fileList) {
                console.log(file, fileList);
            },
            handlePreview(file) {
                console.log(file);
            },
            edit(row) {
                //拷贝后面对象的值到新对象,防止后面代码改动引起模型变化
                let courseTemp = Object.assign({}, row);
                console.log(this.course.detail.intro)
                //回显数据
                this.course = courseTemp;
                //显示
                this.formVisible = true;
                this.getCourseLevels();
                this.getTypeTree();
            },
            save() {//合并了添加和修改
                this.$refs.course.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.saveLoading = true;
                            //拷贝后面对象的值到新对象,防止后面代码改动引起模型变化
                            let para = Object.assign({}, this.course);
                            para.courseTypeId = para.courseTypeId[para.courseTypeId.length-1]
                            console.log(para);
                            console.log("1111111111");
                            this.$http.put("/course/course/", para).then((res) => {
                                this.$message({
                                    message: '操作成功!',
                                    type: 'success'
                                });
                                //关闭刷新框
                                this.saveLoading = false;
                                //重置表单
                                this.$refs['course'].resetFields();
                                //关闭对话框
                                this.formVisible = false;
                                //刷新数据
                                this.getCourses();
                            })
                        });
                    }
                })
            },
            handleCurrentChange(curentPage) {
                this.page = curentPage;
                this.getCourses();
            },
        },
        mounted() {
            this.getCourses()
        }
    }

</script>

<style scoped>

</style>