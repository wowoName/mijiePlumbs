<template>
    <div class='tblmain'>
        <el-form :inline="true" :model="serverForm" size="mini" class="demo-form-inline">
            <el-form-item label="服务id">
                <el-input v-model="serverForm.serverID" placeholder="请输入服务ID"></el-input>
            </el-form-item>

            <el-form-item>
                <el-button type="primary" icon="el-icon-search" @click="onSubmit">查询</el-button>
            </el-form-item>

            <el-form-item>
                <el-button type="success" icon="el-icon-plus" @click="addServe">新增</el-button>
            </el-form-item>

        </el-form>

        <el-table :data="tblData" border size="medium" :row-style="getRowClass" :header-row-style="getRowClass" :header-cell-style="getRowClass" row-key="id" :show-header="true" style="width: 100%; margin-top: 20px">
            <el-table-column prop="id" label="ID" width="180">
            </el-table-column>
            <el-table-column prop="ip" label="IP">
            </el-table-column>
            <el-table-column prop="port" label="端口">
            </el-table-column>
            <el-table-column prop="remark" label="备注">
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页信息 -->
        <div class="pageSizeCon" v-show="tblData.length>10">
            <el-pagination background layout="prev, pager, next" :total="tblData.length" @current-change="getData">
            </el-pagination>
        </div>

        <!-- 删除dialog -->
        <el-dialog title="提示" :visible.sync="deleteInfo" width="360px">
            <span>确定要删除吗?</span>
            <span slot="footer" class="dialog-footer">
                <el-button size="mini" @click="deleteInfo = false">取 消</el-button>
                <el-button size="mini" type="primary" @click="handleDel">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>

export default {
    components: {},
    data() {
        return {
            serverForm: { //查询
                serverID: ""
            },
            //表格数据
            tblData: [{
                id: "1",
                ip: "192.68.1.1",
                port: "8080",
                remark: "新增测试", children: [{
                    id: "1.1",
                    ip: "192.68.1.3",
                    port: "8080",
                    remark: "新增测试"
                }]
            }, {
                id: "2",
                ip: "192.68.1.2",
                port: "8080",
                remark: "新增测试", children: [{
                    id: "2.1",
                    ip: "192.68.1.3",
                    port: "8080",
                    remark: "新增测试"
                }]
            }, {
                id: "3",
                ip: "192.68.1.3",
                port: "8080",
                remark: "新增测试", children: [{
                    id: "3.1",
                    ip: "192.68.1.3",
                    port: "8080",
                    remark: "新增测试"
                }]
            }, {
                id: "4",
                ip: "192.68.1.4",
                port: "8080",
                remark: "编辑",
                children: [{
                    id: "4.1",
                    ip: "192.68.1.3",
                    port: "8080",
                    remark: "编辑"
                }]
            }],
            fanceName: "新增",
            deleteInfo: false//删除dialog
        };
    },
    computed: {},
    watch: {},
    methods: {
        getData(page) {
            console.log(page);
        },
        getRowClass({ row, column, rowIndex, columnIndex }) {
            return "background:#ffffff;color:#000;font-weight:500";
        },
        /**
         * 根据serverId 查询
         */
        onSubmit() {
            console.log(this.serverForm.serverID);
        },
        /**
         * 新增服务
         */
        addServe() {
            this.$router.push("/mijie-pro");
        },
        /** 
         * 编辑
         * @prams {Number} index 当前表格索引
         * @param {Object} data 当前行数据
        */
        handleEdit(index, data) {
            console.log("编辑");
            this.$router.push("/mijie-pro");
        },
        /** 
         * 删除
         * @param {Number} index 当前表格索引
         * @param {Object} data 当前行数据
        */
        handleDelete(index, data) {
            this.deleteInfo = true;
        },
        /** 
         * 删除diaolog回调
        */
        handleDel() {
            console.log("执行删除");
            this.$message({
                message: '删除成功!',
                type: 'success'
            });
            this.deleteInfo = false;
        },
        doSearchFaceList(callback) {
            console.log(callback);
            return this.tblData.map(callback)
        },
    },
    created() {

    },
    mounted() {

    },
    beforeCreate() { }, //生命周期 - 创建之前
    beforeMount() { }, //生命周期 - 挂载之前
    beforeUpdate() { }, //生命周期 - 更新之前
    updated() { }, //生命周期 - 更新之后
    beforeDestroy() { }, //生命周期 - 销毁之前
    destroyed() { }, //生命周期 - 销毁完成
    activated() { }, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style lang='scss' scoped>
.tblmain {
    padding: 5px 8px;
    box-sizing: border-box;

    .pageSizeCon {
        margin-top: 16px;
    }

    .warning-row {
        background: oldlace;
    }
    .success-row {
        background: #f0f9eb;
    }
}
.dialogClass {
    padding: 0;
}
</style>