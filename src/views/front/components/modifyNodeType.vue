<template>
    <div>
        <el-form :model="modifyForm" :rules="rules" ref="modifyForm" label-width="110px" class="demo-ruleForm">
            <el-form-item :label="$t('nodes.admin')" prop="adminRivateKey" style="width: 320px;">
                <el-select v-model="modifyForm.adminRivateKey" :placeholder="$t('text.select')">
                    <el-option v-for="(item,index) in adminRivateKeyList" :key="index" :label="item.userName" :value="item.address">
                        <span>{{item.userName}}</span>
                        <span class="font-12">{{item.address | splitString}}</span>
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item :label="$t('nodes.nodeStyle')" prop="nodeType" style="width: 320px;">
                <el-select v-model="modifyForm.nodeType"  @change="weightIfShow" :placeholder="$t('text.select')">
                    <el-option v-for="item in nodeTypeList" :key="item.type" :label="item.name" :value="item.type">
                    </el-option>
                </el-select>
            </el-form-item>
             <el-form-item :label="$t('govCommittee.weight')" prop="weight" style="width: 320px;" v-if="weightShow">
                <el-input v-model="modifyForm.weight" :placeholder="$t('text.select')">
            
               
                </el-input>
            </el-form-item>
            <p class="info" v-if='deployType == 1'>{{$t('nodes.notice')}}</p>
        </el-form>
        <div class="text-right sure-btn" style="margin-top:10px">
            <el-button @click="close">{{this.$t("text.cancel")}}</el-button>
            <el-button type="primary" :loading="loading" @click="submit('modifyForm')">{{this.$t("text.sure")}}</el-button>
        </div>
    </div>
</template>

<script>
import { getUserList, consensusNodeId } from "@/util/api";
export default {
    name: 'ModifyNodeType',

    components: {
    },

    props: {
        modifyNode: {
            type: Object
        }
    },

    data() {
        return {
            weightShow:false,
            loading: false,
            adminRivateKeyList: [],
            nodeTypeList: [
                {
                    type: 'sealer',
                    name: this.$t("nodes.sealer")
                },
                {
                    type: 'observer',
                    name: this.$t("nodes.observer")
                },
                {
                    type: 'remove',
                    name: this.$t("nodes.remove")
                },
            ],
            modifyForm: {
                adminRivateKey: this.modifyNode.adminRivateKey,
                nodeType: this.modifyNode.nodeType,
                weight:this.modifyNode.weight==0?1:this.modifyNode.weight,
            },
            deployType: null,
            ruleTest: this.$t("rule.adminRule")
        }
    },
    computed: {
        rules() {
            var validateWeight = (rule, value, callback) => {
                if (value == '' || value == undefined || value == null) {
                    callback();
                } else {
                    if (Number(value)==0) {
                          callback(new Error(this.$t('rule.blockNumber')));
                    } else if(!Number(value)){
                        callback(new Error(this.$t('rule.inputIsNumber')));
                    }else {
                        if (value <= 0||value>=2147483647) {
                            callback(new Error(this.$t('rule.weightNumber')));
                        } else {
                            callback();
                        }
                    }
                }
            }
            let data = {
                adminRivateKey: [
                    {
                        required: true,
                        message: this.ruleTest,
                        trigger: "blur"
                    }
                ],
                nodeType: [
                    {
                        required: true,
                        message: this.$t("rule.nodeType"),
                        trigger: "blur"
                    }
                ],
                 weight: [
                    {
                        required: true,
                        message: this.$t("rule.weightRule"),
                        trigger: "blur"
                    },
                     {
                        required: true,
                        validator:validateWeight,
                        trigger: "blur"
                    }
                ]
            }
            return data
        }
    },

    watch: {

    },

    created() {

    },

    mounted() {
        if (localStorage.getItem("deployType")) {
            this.deployType = localStorage.getItem("deployType")
        } else {
            this.deployType = 0
        }
        this.modifyNode.nodeType === 'sealer'?this.weightShow=true:this.weightShow=false
        this.getUserData();
    },

    methods: {
        weightIfShow() {
                this.modifyForm.nodeType === 'sealer'?this.weightShow=true:this.weightShow=false
        },
        close() {
            this.$emit("nodeModifyClose");
        },
        submit(formName) {
            this.$refs[formName].validate(valid => {
                if (valid) {
                    if (this.modifyNode.nodeType === 'sealer' && this.modifyForm.nodeType === 'observer') {
                        this.$confirm(this.$t("nodes.observerText"), this.$t("text.tips"), {
                            confirmButtonText: this.$t("text.sure"),
                            cancelButtonText: this.$t("text.cancel"),
                            type: 'warning'
                        }).then(() => {
                            this.queryConsensusNodeId()
                        }).catch(() => {
                            console.log('close')
                        });
                    } else if ((this.modifyNode.nodeType === 'sealer' && this.modifyForm.nodeType === 'remove') || (this.modifyNode.nodeType === 'observer' && this.modifyForm.nodeType === 'remove')) {
                        this.$confirm(this.$t("nodes.removeText"), this.$t("text.tips"), {
                            confirmButtonText: this.$t("text.sure"),
                            cancelButtonText: this.$t("text.cancel"),
                            type: 'warning'
                        }).then(() => {
                            this.queryConsensusNodeId()
                        }).catch(() => {
                            console.log('close')
                        });
                    } else if ((this.modifyNode.nodeType === 'observer' && this.modifyForm.nodeType === 'sealer') || (this.modifyNode.nodeType === 'remove' && this.modifyForm.nodeType === 'sealer')) {
                        this.$confirm(this.$t("nodes.sealerText"), this.$t("text.tips"), {
                            confirmButtonText: this.$t("text.sure"),
                            cancelButtonText: this.$t("text.cancel"),
                            type: 'warning'
                        }).then(() => {
                            this.queryConsensusNodeId()
                        }).catch(() => {
                            console.log('close')
                        });
                    } else {
                        this.queryConsensusNodeId()
                    }

                } else {
                    return false;
                }
            });

        },
        queryConsensusNodeId() {
            this.loading = true;
            let reqData = {
                groupId: localStorage.getItem("groupId"),
                nodeType: this.modifyForm.nodeType,
                nodeId: this.modifyNode.nodeId,
                fromAddress: this.modifyForm.adminRivateKey,
                weight:this.modifyForm.weight,
            }
            consensusNodeId(reqData)
                .then(res => {
                    this.loading = false;
                    if (res.data.code === 0) {
                        this.$message({
                            type: 'success',
                            message: this.$t("text.updateSuccessMsg")
                        })
                        this.$emit('nodeModifySuccess')
                    } else {
                        this.$message({
                            message: this.$chooseLang(res.data.code),
                            type: "error",
                            duration: 2000
                        });
                    }
                })
                .catch(err => {
                    this.loading = false;
                    this.$message({
                        message: err.data || this.$t('text.systemError'),
                        type: "error",
                        duration: 2000
                    });
                });
        },
        changeRivateKey(val) {
            this.adminRivateKey = val
        },
        getUserData: function () {
            let reqData = {
                groupId: localStorage.getItem("groupId"),
                pageNumber: 1,
                pageSize: 1000
            };
            let query = {}
            if (localStorage.getItem('root') === 'developer') {
                query.account = localStorage.getItem("user")
            }
            getUserList(reqData, query)
                .then(res => {
                    if (res.data.code === 0) {
                        if (res.data.data.length === 0) {
                            this.ruleTest = this.$t("text.ruleAddUser")
                        }
                        this.adminRivateKeyList = [];
                        res.data.data.forEach(value => {
                            if (value.hasPk === 1) {
                                this.adminRivateKeyList.push(value);
                            }
                        });
                        if (this.adminRivateKeyList.length) this.modifyForm.adminRivateKey = this.adminRivateKeyList[0]['address'];
                    } else {
                        this.$message({
                            message: this.$chooseLang(res.data.code),
                            type: "error",
                            duration: 2000
                        });
                    }
                })
                .catch(err => {
                    this.$message({
                        message: err.data || this.$t('text.systemError'),
                        type: "error",
                        duration: 2000
                    });
                });
        },
    }
}
</script>

<style scoped>
.sure-btn >>> .el-button {
    padding: 9px 16px;
}
.info {
    padding-left: 30px;
    color: #f00;
}
</style>
