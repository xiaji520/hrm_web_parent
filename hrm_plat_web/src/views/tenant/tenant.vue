<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :model="filters" :inline="true">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getTenants">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="add">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--
        列表v-loading="listLoading"
        private Long pid;
        -->
        <el-table :data="tenants" v-loading="listLoading" highlight-current-row style="width: 100%;">
            <!--多选框-->
            <el-table-column type="selection" width="55">
            </el-table-column>
            <!--索引值,为什么不用id,id不序号-->
            <el-table-column type="index" width="60">
            </el-table-column>
            <!--其他都设置值,只有一个不设置值就自动适应了-->
            <el-table-column prop="logo" label="图标">
            </el-table-column>
            <el-table-column prop="companyName" label="机构名称">
            </el-table-column>
            <el-table-column prop="companyNum" label="机构编号">
            </el-table-column>
            <el-table-column prop="registerTime" label="注册时间">
            </el-table-column>
            <el-table-column prop="state" label="状态" :formatter="formatState">
            </el-table-column>
            <el-table-column prop="address" label="地址">
            </el-table-column>
            <el-table-column prop="tenantType" label="机构类型" :formatter="formatTenantType">

            </el-table-column>
            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="edit(scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="del(scope.row)">删除</el-button>
                </template>
            </el-table-column>

        </el-table>
        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="10"
                           :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--添加或编辑对话框-->
        <el-dialog title="添加或修改" :visible.sync="formVisible" :close-on-click-modal="false">
            <el-form :model="tenant" label-width="80px" :rules="formRules" ref="tenant">
                <el-form-item label="图标" prop="logo">
                    <el-input v-model="tenant.logo" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="机构名称" prop="companyName">
                    <el-input v-model="tenant.companyName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="机构编号" prop="companyNum">
                    <el-input v-model="tenant.companyNum" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="状态" prop="state">
                    <el-radio-group v-model="tenant.state">
                        <el-radio checked="checked" :label="0">已启用</el-radio>
                        <el-radio :label="1">未启用</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="地址" prop="address">
                    <el-input v-model="tenant.address" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="机构类型" prop="tenantType">
                    <el-input v-model="tenant.tenantType" auto-complete="off"></el-input>
                </el-form-item>
                <!--<el-form-item label="机构类型" prop="manager">
                    <el-select v-model="tenant.manager" placeholder="请选择">
                        <el-option
                                v-for="item in employees"
                                :key="item.id"
                                :label="item.username"
                                :value="item.id">
                            <span style="float: left">{{ item.id }}</span>
                            <span style="float: right; color: #8492a6; font-size: 13px">{{ item.username }}</span>
                        </el-option>
                    </el-select>
                </el-form-item>-->
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="formVisible = false">取消</el-button>
                <el-button type="primary" @click="save">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    export default {
        data() {
            return {
                tenantTypes: [],
                formVisible: false,//对话框默认不显示,只有点击添加或修改的时候显示
                listLoading: false,
                //查询对象
                filters: {
                    keyword: ''
                },
                page: 1,//当前页,要传递到后台的
                total: 0, //分页总数
                tenants: [], //当前页数据
                //初始值
                tenant: {
                    id: null,
                    logo: '',
                    companyName: '',
                    companyNum: '',
                    state: 0,
                    address: '',
                    tenantType: ''
                },
                employees: [],
                formRules: {
                    /* name: [
                         {required: true, message: '请输入名称!', trigger: 'blur'}
                     ]*/
                }
            }
        },
        methods: {
            getTenantTypes() {
                this.$http.patch("/sysmanage/tenantType/")
                    .then(result => {
                        this.tenantTypes = result.data
                    });
            },
            add() {
                //清空数据
                this.tenant = {
                    id: null,
                    logo: '',
                    companyName: '',
                    companyNum: '',
                    state: 0,
                    address: '',
                    tenantType: ''
                }
                //打开dialog
                this.formVisible = true;
            },
            handleCurrentChange(curentPage) {
                this.page = curentPage;
                this.getTenants();
            },
            save() {
                this.$refs.tenant.validate((valid) => {
                    //校验表单成功后才做一下操作
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            //拷贝后面对象的值到新对象,防止后面代码改动引起模型变化
                            let para = Object.assign({}, this.tenant);
                            //判断是否有id有就是修改,否则就是添加
                            this.$http.put("/sysmanage/tenant", para).then((res) => {
                                this.$message({
                                    message: '操作成功!',
                                    type: 'success'
                                });
                                //重置表单
                                this.$refs['tenant'].resetFields();
                                //关闭对话框
                                this.formVisible = false;
                                //刷新数据
                                this.getTenants();
                            });
                        });
                    }
                })
            },
            edit(row) {
                //回显
                let tenantTmp = Object.assign({}, row); //解决对话框改值后列表会被改值.
                if (tenantTmp.state === true) {
                    tenantTmp.state = 0
                } else if (tenantTmp.state === false) {
                    tenantTmp.state = 1
                }
                this.tenant = tenantTmp; //里面本来就有id,相当于回显了id
                //显示
                this.formVisible = true;
            },
            getTenants() {
                //发送Ajax请求后台获取数据  axios
                //添加分页条件及高级查询条件
                let para = {
                    "page": this.page,
                    "keyword": this.filters.keyword
                };
                this.listLoading = true; //显示加载圈
                //分页查询
                this.$http.post("/sysmanage/tenant/", para) //$.Post(.....)
                    .then(result => {
                        this.total = result.data.total;
                        this.tenants = result.data.rows;
                        console.log(result.data.rows)
                        this.listLoading = false;  //关闭加载圈
                    });
            },
            del(row) {
                console.log(row);
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    var id = row.id;
                    this.listLoading = true;
                    this.$http.delete("/sysmanage/tenant/" + id)
                        .then(result => {
                            this.listLoading = false;
                            //做提示
                            if (result.data.success) {
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                            } else {
                                this.$message({
                                    message: result.data.message,
                                    type: 'error'
                                });
                            }
                            //刷新数据
                            this.getTenants();
                        })
                });

            },
            //初始化是否启用
            formatState(row) {
                return row.state === true ? '已启用' : row.state === false ? '未启用' : '未知状态';
            },
            //初始化机构类型
            formatTenantType(row) {
                //console.log("row.tenantType:"+row.tenantType)
                for (var i in this.tenantTypes) {
                    //console.log(this.tenantTypes[i].id);
                    if (this.tenantTypes[i].id == row.tenantType) {
                        //console.log(this.tenantTypes[i].name)
                        return this.tenantTypes[i].name
                    }else if (row.tenantType==null) {
                        return "未知类型"
                    }
                }
            }
        },
        mounted() {
            this.getTenants()
            this.getTenantTypes()
        }
    }

</script>

<style scoped>

</style>