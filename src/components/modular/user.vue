<template>
    <div class="__resident">
        <div class="formBox">
            <el-form :inline="true" ref="formInline" :model="formInline" class="demo-form-inline">
                <el-form-item label="姓名：">
                    <el-input v-model="formInline.real_name" placeholder="请输入姓名"></el-input>
                </el-form-item>
                <el-form-item label="手机号码：">
                    <el-input v-model="formInline.phone" placeholder="请输入手机号码"></el-input>
                </el-form-item>
                <el-form-item label="性别：">
                    <el-select v-model="formInline.sex" placeholder="请选择性别">
                        <el-option label="男" value="1"></el-option>
                        <el-option label="女" value="2"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" icon="el-icon-search" @click="searchSubmit('formInline')">查询</el-button>
                </el-form-item>
            </el-form>
        </div>
        <div class="btnBox">
            <el-button type="primary" icon="el-icon-circle-plus-outline" @click="onClickAdd">新增居民</el-button>
            <el-button type="primary" icon="el-icon-upload2" @click="addByImport">导入</el-button>
            <el-button type="primary" icon="el-icon-download" @click="queryByExport">导出</el-button>
        </div>
        <div class="tableList">
            <vtable :dataArray="dataArray" :columns="columns" :total="total" :isSelection="isSelection" @getArticle="queryUserListPost"></vtable>
        </div>

        <el-dialog title="权限设置" :visible.sync="dialogRoleVisible" :modal-append-to-body="false"
               :close-on-click-modal="false">
            <div class="modelFromListBox">
                <el-upload class="upload-demo" ref="upload" :action="action" :on-preview="handlePreview"
                        :on-remove="handleRemove" :file-list="fileList" :on-success="handleAvatarSuccess"
                        :before-upload="beforeAvatarUpload" :auto-upload="false">
                    <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
                </el-upload>
            </div>
            <div slot="footer" class="dialog-footer">
                <el-button @click="resetRole()">取 消</el-button>
                <el-button type="primary" @click="submitRole()">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import { path, uploadXls } from '@/path/path'
import table from '@/components/common/table'
import MyDropDown from '@/components/common/MyDropDown'
import { getCookie, setCookie } from "@/util/cookie";
import { queryResidentList, upadteResident } from "api/resident/index";
import { exportExcel, importExcel } from "api/upload/index";
export default {
    components: {
        vtable: table
    },
    data() {
        return {
            dialogRoleVisible: false,//菜单权限弹窗
            fileList: [],
            action: uploadXls,
            formInline: {
                real_name: '',
                phone: '',
                sex: '',
                role: ''
            },
            dataArray: [],
            pageSize: 10,
            pageNum: 1,
            total: 0,
            isSelection: false,
            columns: [
                {
                    prop: "name",
                    label: "姓名",
                },
                {
                    prop: "sex",
                    label: "性别",
                    render: function(createElement) {
                        if(this.row.sex==1){
                            return createElement('span', {
                                domProps: {
                                    innerHTML: '男',
                                }
                            })
                        }else if(this.row.sex==2){
                            return createElement('span', {
                                domProps: {
                                    innerHTML: '女',
                                }
                            })
                        }
                    },
                },
                {
                    prop: "phone",
                    label: "手机号码",
                },
                {
                    prop: "card_number",
                    label: "身份证号码",
                },
                {
                    prop: "",
                    label: "操作",
                    render: (h, param) => {
                        const dropDownData = {
                            label: "操作",
                            items: [{
                                    label: "修改",
                                    func: {
                                        func: "update",
                                        uuid: param.row.uuid
                                    }
                                },
                            ]
                        };
                        return h(MyDropDown, {
                            props: {
                                dropDownData: dropDownData
                            },
                            on: {
                                update: this.update,
                            }
                        });
                    }
                }
            ],
        }
    },
    mounted() {
        this.queryUserListPost(this.pageNum);
    },
    methods: {
        //搜索
        searchSubmit(formName) {
            this.queryUserListPost(this.pageNum, this.formInline.real_name, this.formInline.phone, this.formInline.sex, this.formInline.role,);
        },
        //新增
        onClickAdd() {
            this.$router.push({path: '/home/userAdd?type=1' })
        },
        //导入
        addByImport(){
            this.dialogRoleVisible = true
        },
        //导出
        queryByExport(){
            window.location.href = path + 'excel/exportExcel.do'
        },
        //修改
        update(obj){
            this.$router.push({path: '/home/userAdd?uuid='+obj+'&type=2' })
        },
        //查询列表
        queryUserListPost(pageNum, real_name, phone, sex, role) {
            let params = {
                pageSize: this.pageSize,
                pageNum: pageNum,
                name: real_name,
                phone: phone,
                sex: sex,
            };
            queryResidentList(params).then(data => {
                if(data.data.status==200){
                    this.total = data.data.total
					this.dataArray = data.data.data
                }
            })
        },


        //导入
        addByImport() {
            this.dialogRoleVisible = true
        },
        //取消弹窗
        resetRole() {
            this.dialogRoleVisible = false
        },
        //新增弹窗
        submitRole() {
            this.$refs.upload.submit();
        },
        submitUpload() {
            this.$refs.upload.submit();
        },
        handleRemove(file, fileList) {
            console.log(file, fileList);
        },
        handlePreview(file) {
            console.log(file);
        },
        // 上传图片前
        beforeAvatarUpload(file) {
            const isLt2M = file.size / 1024 / 1024 < 10;
            if (!isLt2M) {
            this.$message.error('上传头像图片大小不能超过 10MB!');
            }
            return isLt2M;
        },
        handleAvatarSuccess(res, file) {
            importExcel(file.raw).then(data => {
                if(data.data.status==200){
                    this.$message({
                        message: '导入成功',
                        type: 'success',
                        duration: '500',
                        onClose: function(){
                            window.location.reload();
                        }
                    });
                }
            })
        },
    }
}
</script>

<style lang="stylus">
.__resident {
    background #ffffff;
    padding 15px;
    box-shadow: 0 1px 2px 0 rgba(0,0,0,0.05);
    border-radius: 4px;
    .formBox {
        border-bottom 1px solid #eee
    }
    .btnBox {
        margin 10px 0;
    }
}
</style>