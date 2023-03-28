<template>
    <div>
        <div class="search-cont">
            <el-form :inline="true" :class="isSearchCollapse ? 'more' : ''" class="zui-form" ref="searchForm"
                :model="searchForm" @keyup.enter.native="pageNum = 1; refreshList()" @submit.native.prevent>
                <!-- 搜索框-->
                <el-form-item prop="year">
                    <el-input size="small" v-model="searchForm.year" placeholder="预算年度" clearable></el-input>
                </el-form-item>
                <el-form-item prop="subControl">
                    <el-select v-model="searchForm.subControl" placeholder="所属分控" size="small" style="width: 100%;"
                        clearable>
                        <el-option v-for="item in $dictUtils.getDictList('sub_control').filter(it => it.value !== '9')"
                            :key="item.value" :label="item.label" :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item prop="officeName">
                    <el-input size="small" v-model="searchForm.officeName" placeholder="填报单位" clearable></el-input>
                </el-form-item>
                <el-form-item prop="categoryName">
                    <el-input size="small" v-model="searchForm.categoryName" placeholder="资产分类" clearable></el-input>
                </el-form-item>
                <el-form-item prop="assetName">
                    <el-input size="small" v-model="searchForm.assetName" placeholder="资产名称" clearable></el-input>
                </el-form-item>
                <el-form-item prop="fundsFrom">
                    <el-select v-model="searchForm.fundsFrom" placeholder="资金来源" size="small" style="width: 100%;"
                        clearable>
                        <el-option v-for="item in $dictUtils.getDictList('funds_from')" :key="item.value"
                            :label="item.label" :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item prop="companyName">
                    <el-input size="small" v-model="searchForm.companyName" placeholder="需求单位" clearable></el-input>
                </el-form-item>
                <el-form-item prop="needUserName">
                    <el-input size="small" v-model="searchForm.needUserName" placeholder="需求提报人" clearable></el-input>
                </el-form-item>
            </el-form>
            <div class="zui-operate">
                <div class="zo-item b" title="搜索" @click="pageNum = 1; refreshList()"><i class="el-icon-search"></i>
                </div>
                <div class="zo-item" title="重置" @click="resetSearch()"><i class="el-icon-refresh-left"></i></div>
                <div class="zo-item" title="刷新" @click="refreshList"><i class="el-icon-refresh"></i></div>
            </div>
            <div class="moreSearch" :class="isSearchCollapse ? 'close' : ''"
                @click="isSearchCollapse = !isSearchCollapse, isImportCollapse = false">
                <p class="text">{{ isSearchCollapse ? '收起' : '高级搜索' }}</p>
                <div class="icon"><i class="el-icon-d-arrow-right"></i></div>
            </div>
        </div>
        <el-dialog title="导入" :visible.sync="importVisible" v-if="importVisible" width="30%">

            <el-upload class="upload-demo" accept=".xls,.xlsx"
                :action="`${this.$http.BASE_URL}/data/daBudgetRecord/importFile`" :on-success="uploadSuccess"
                :show-file-list="true">
                <el-button size="small" type="danger">点击上传</el-button>
                <div slot="tip" class="el-upload__tip">只允许导入“xls”或“xlsx”格式文件！</div>
            </el-upload>
            <span slot="footer" class="dialog-footer">
                <el-button @click="importVisible = false">取 消</el-button>
            </span>
        </el-dialog>
        <el-row class='operate-cont'>
            <el-button type="info" icon="el-icon-download" @click="$utils.download('/data/daBudgetRecord/exportTemplate')
            " size="medium" title="下载模板">下载模板</el-button>
            <el-button type="danger" @click="importVisible = true;" size="medium" title="导入数据">导入数据</el-button>
            <el-button type="danger" :disabled="dataListSelections.length <= 0" @click="del()" size="medium"
                title="批量删除">批量删除</el-button>
        </el-row>
        <div class="table-box">
            <el-table :data="dataList" border stripe size="medium" @selection-change="selectionChangeHandle"
                @sort-change="sortChangeHandle" v-loading="loading" class="table" height="530">
                <el-table-column type="selection" key="101" header-align="center" align="center" width="60">
                </el-table-column>
                <el-table-column key="102" type="index" label="序号" header-align="center" align="center" width="60">
                </el-table-column>
                <el-table-column key="103" prop="period" show-overflow-tooltip label="预算年度" min-width="120">
                </el-table-column>
                <el-table-column key="107" prop="subControl" show-overflow-tooltip label="所属分控" min-width="120">
                    <template slot-scope="scope">
                        {{ $dictUtils.getDictLabel("sub_control", scope.row.subControl, '-') }}
                    </template>
                </el-table-column>
                <el-table-column key="1107" prop="originateOrgName" show-overflow-tooltip label="填报单位" min-width="160">
                </el-table-column>
                <el-table-column key="1108" prop="fillOrgName" show-overflow-tooltip label="需求单位" min-width="160">
                </el-table-column>
                <el-table-column key="108" prop="fundsFrom" show-overflow-tooltip label="资金来源" min-width="120">
                    <template slot-scope="scope">
                        {{ $dictUtils.getDictLabel("funds_from", scope.row.fundsFrom, '-') }}
                    </template>
                </el-table-column>
                <el-table-column key="106" prop="categoryName" show-overflow-tooltip label="资产分类" min-width="120">
                </el-table-column>
                <el-table-column key="109" prop="assetName" show-overflow-tooltip label="资产名称" min-width="240">
                </el-table-column>
                <el-table-column prop="orgName" show-overflow-tooltip label="所在部门" min-width="240">
                </el-table-column>
                <el-table-column key="113" prop="brand" show-overflow-tooltip label="品牌" min-width="120">
                </el-table-column>
                <el-table-column key="114" prop="spec" show-overflow-tooltip label="规格" min-width="120">
                </el-table-column>
                <el-table-column key="115" prop="model" show-overflow-tooltip label="型号" min-width="120">
                </el-table-column>
                <el-table-column key="110" prop="number" show-overflow-tooltip label="需求数量" min-width="120">
                </el-table-column>
                <el-table-column key="111" prop="numberUnit" show-overflow-tooltip label="数量单位" min-width="120">
                    <template slot-scope="scope">
                        {{ $dictUtils.getDictLabel("number_unit", scope.row.numberUnit, '-') }}
                    </template>
                </el-table-column>
                <el-table-column key="112" prop="funds" show-overflow-tooltip label="预估总价(元)" min-width="120">
                    <template slot-scope="scope">
                        {{ fmoney(scope.row.funds) }}
                    </template>
                </el-table-column>
                <el-table-column key="117" prop="purpose" show-overflow-tooltip label="用途" min-width="240">
                </el-table-column>
                <el-table-column key="116" prop="needUserName" show-overflow-tooltip label="需求提报人" min-width="120">
                </el-table-column>
                <el-table-column key="118" prop="remarks" show-overflow-tooltip label="备注" min-width="240">
                </el-table-column>
                <!-- <el-table-column key="104" prop="type" show-overflow-tooltip label="计划类型" min-width="120">
                    <template slot-scope="scope">
                        {{ $dictUtils.getDictLabel("plan_type", scope.row.type, '-') }}
                    </template>
                </el-table-column>
                <el-table-column key="105" prop="companyName" show-overflow-tooltip label="需求单位" min-width="240">
                </el-table-column>
                <el-table-column prop="idle" show-overflow-tooltip label="同类资产闲置数" min-width="120">
				</el-table-column> -->
                <el-table-column key="119" header-align="center" align="center" fixed="right" width="130" label="操作">
                    <template slot-scope="scope">
                        <el-button key="137" type="text" size="small" @click="del(scope.row.id)"
                            class="icon iconfont icon-shanchu" title="删除"></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination key="120" @size-change="sizeChangeHandle" @current-change="currentChangeHandle"
                :current-page="pageNum" :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="total" background
                layout="total, sizes, prev, pager, next, jumper">
            </el-pagination>
        </div>
        <importTable ref="importTable"></importTable>
    </div>
</template>

<script>
import importTable from '@/components/importTable/index'
export default {
    data() {
        return {
            searchForm: {
                budgetPlanId: this.budgetPlanId,
                budgetTaskId: this.budgetTaskId,
                budgetTaskRecordId: this.budgetTaskRecordId,
                budgetApproveId: this.budgetApproveId,
                budgetType: '1',
                code: '',
                year: '',
                fundsFrom: '',
                subControl: '',
                categoryId: '',
                companyId: '',
                companyName: '',
                categoryName: '',
                assetName: '',
                officeId: '',
                officeName: '',
                orgId: '',
                needUserId: '',
                needUserName: '',
                useUserId: '',
                numberUnit: '',
                purchaseDate: '',
                reviseDelFlag: '0',
                statusNotList: this.containDraft ? [] : ['0']
            },
            dataList: [],
            pageNum: 1,
            pageSize: 10,
            total: 0,
            orderBy: '',
            dataListSelections: [],
            importVisible: false,
            isSearchCollapse: false,
            isImportCollapse: false,
            loading: false
        }
    },
    components: {
        importTable
    },
    mounted() {
        this.refreshList()
    },

    methods: {
        // 导入成功
        uploadSuccess(res, file) {
            console.log('res', res)
            this.$refs.importTable.init(res.body)
            if (res.success) {
                this.importVisible = false
                this.refreshList()
                this.$message.success({
                    dangerouslyUseHTMLString: true,
                    message: res.msg
                })
            } else {
                if (res.body.errorList && res.body.errorList.length) {
                    this.$refs.importTable.init(res.body)
                } else {
                    this.$message.error({
                        dangerouslyUseHTMLString: true,
                        message: res.msg
                    })
                }
            }
            this.uploadLoding ? this.uploadLoding.close() : ''
        },
        // 获取数据列表
        refreshList() {
            this.loading = true
            this.$http({
                url: '/data/daBudgetRecord/findPage',
                method: 'post',
                data: {
                    'pageNum': this.pageNum,
                    'pageSize': this.pageSize,
                    value1: '1',
                    'orderBy': this.orderBy,
                    beginPurchaseDate: this.searchForm.purchaseDate[0],
                    endPurchaseDate: this.searchForm.purchaseDate[1],
                    ...this.lodash.omit(this.searchForm, 'purchaseDate')
                }
            }).then(({ data }) => {
                if (data && data.success) {
                    this.dataList = data.body.result.list
                    this.total = data.body.result.total
                    this.loading = false
                }
            })
        },
        // 每页数
        sizeChangeHandle(val) {
            this.pageSize = val
            this.pageNum = 1
            this.refreshList()
        },
        // 当前页
        currentChangeHandle(val) {
            this.pageNum = val
            this.refreshList()
        },
        // 多选
        selectionChangeHandle(val) {
            this.dataListSelections = val
        },
        // 排序
        sortChangeHandle(obj) {
            if (obj.order === 'ascending') {
                this.orderBy = obj.prop + ' asc'
            } else if (obj.order === 'descending') {
                this.orderBy = obj.prop + ' desc'
            } else {
                this.orderBy = ''
            }
            this.refreshList()
        },
        // 删除
        del(id) {
            let ids = id || this.dataListSelections.map(item => item.id).join(',')
            this.$confirm(`确定删除所选项吗?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.loading = true
                this.$http({
                    url: '/data/daBudgetRecord/deleteAll',
                    method: 'post',
                    data: { 'ids': ids, ...this.searchForm, value1: '1', }
                }).then(({ data }) => {
                    if (data && data.success) {
                        this.$message.success(data.msg)
                        this.refreshList()
                    }
                    this.loading = false
                })
            })
        },
        // 导入成功
        uploadSuccess(res, file) {
            if (res.success) {
                this.$message.success({
                    dangerouslyUseHTMLString: true,
                    message: res.msg
                })
            } else {
                this.$message.error(res.msg)
            }
        },
        resetSearch() {
            this.$refs.searchForm.resetFields()
            this.refreshList()
        }
    }
}
</script>
