<template>
    <div class="modify">
        <el-dialog title="修改密码" :visible.sync="modifyPasswordModal.showDialog" width='416px' :close-on-click-modal='false'>
            <el-form :model="modifyForm" status-icon :rules="rules" ref="modifyForm" label-width="70px"
                class="demo-ruleForm" label-position="left">
                <el-form-item label="" prop="" class="hiddenItem">
                    <!-- 假的控件，用来填充，兼容360浏览器 -->
                    <input type="password" name='pass' disabled style='visibility: hidden;'/>
                </el-form-item>
                <el-form-item label="旧密码" prop="oldPwd">
                    <el-input prefix-icon="el-icon-lock" name='pass' v-model="modifyForm.oldPwd" type="password" placeholder="旧密码" autocomplete="new-password"
                        @keyup.enter.native="enterNext1"></el-input>
                </el-form-item>
                <el-form-item label="新密码" prop="newPwd">
                    <el-input ref="newPwd" prefix-icon="el-icon-lock" type="password" placeholder="新密码"
                        v-model="modifyForm.newPwd" @keyup.enter.native="enterNext2"></el-input>
                </el-form-item>
                <el-form-item label="确认密码" prop="confirmPwd">
                    <el-input prefix-icon="el-icon-lock" ref="confirmPwd" type="password" placeholder="确认密码"
                        v-model="modifyForm.confirmPwd" @keyup.enter.native="submitmodifyForm('modifyForm')">
                    </el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="modifyPasswordModal.showDialog = false">取 消</el-button>
                <el-button type="primary" @click="submitmodifyForm('modifyForm')">确认</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        props: ['modifyPasswordModal'],
        data() {
            return {
                modifyForm: {
                    oldPwd: '',
                    newPwd: '',
                    confirmPwd: ''
                },
                rules: {
                    oldPwd: [
                        {
                            required: true,
                            message: "请输入旧密码",
                            trigger: "blur"
                        }
                    ],
                    newPwd: [
                        {
                            required: true,
                            message: "请输入新密码",
                            trigger: "blur"

                        },
                        {
                          trigger: 'blur',
                            validator: (rule, value, callback) => {
                                var passwordreg = /(?=.*?[A-Z])(?=.*\d)(?=.*?[a-z]){6,}/
                                if (!passwordreg.test(value)) {
                                    callback(new Error('新密码必须由数字、字母大小写组合且长度不低于6位'))
                                }else{
                                    callback()
                                }
                            } 
                        }
                    ],
                    confirmPwd: [
                        {
                            required: true,
                            message: "请输入确认密码",
                            trigger: "blur"
                        },
                        {
                          trigger: 'blur',
                            validator: (rule, value, callback) => {
                                var passwordreg = /(?=.*?[A-Z])(?=.*\d)(?=.*?[a-z]){6,}/
                                if (!passwordreg.test(value)) {
                                    callback(new Error('密码必须由数字、字母大小写组合且长度不低于6位'))
                                }else{
                                    callback()
                                }
                            } 
                        }
                    ]
                }
            }
        },
        components: {},
        computed: {},
        created() {
            this.modifyForm.oldPwd = ''
        },
        mounted() { },
        methods: {
            // 确认修改密码
            submitmodifyForm(formName) {
                this.$refs[formName].validate(valid => {
                    if (valid) {
                        let user = { idUserAccount: localStorage.getItem('idUserAccount') }
                        let newObj = Object.assign({}, user, this.modifyForm);
                        newObj.oldPwd = this.$md5(newObj.oldPwd);
                        newObj.newPwd = this.$md5(newObj.newPwd);
                        newObj.confirmPwd = this.$md5(newObj.confirmPwd);
                        this.$axios.post('/upm/account/updPwd', newObj).then(res => {
                            if (res.success) {
                                this.$message({
                                    type: "success",
                                    message: res.message
                                })
                                this.modifyPasswordModal.showDialog = false
                                this.comeRouter({
                                    path: '/login'
                                })
                            } else {
                                this.$message({
                                    type: "warning",
                                    message: res.message
                                })
                            }
                        })
                    }
                })
            },
            enterNext1() {
                this.$refs.newPwd.focus();
            },
            enterNext2() {
                this.$refs.confirmPwd.focus();
            }
        }
    }

</script>
<style scoped>
    .modify {
        width: 336px;
    }
    .el-dialog__wrapper /deep/ .el-dialog__header {
        border-bottom: 1px solid #DCDFE6;
    }
    .el-dialog__wrapper /deep/ .el-dialog__body {
        padding: 20px 20px 0 20px;
    }
    .el-form {
        margin-top: -50px;
    }
    .hiddenItem {
        width: 20%;
    }
</style>