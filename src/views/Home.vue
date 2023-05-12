<template>
    <div class="table-box">
        <div class="title">
            <h2>最简单的 CRUD Demo</h2>
        </div>

        <!-- query -->
        <div class="query-box">
            <el-input class="queryInput" v-model="queryInput" placeholder="请输入姓名搜索" @change="handleQueryName" />
            <div class="btnList">
                <el-button type="primary" @click="handleAdd">增加</el-button>
                <el-button type="danger" @click="handleDelList" v-if = "multipleSelection.length > 0">删除多选</el-button>
            </div>
        </div>
        <!-- table -->      <!-- el-table可用border加边框-->
        <el-table border ref="multipleTableRef" :data="tableData" style="width: 100%"
            @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55" />
            <el-table-column prop="Name" label="姓名" width="120" />
            <el-table-column prop="Email" label="邮箱" width="150" />
            <el-table-column prop="State" label="状态" width="120" />
            <el-table-column prop="Phone" label="电话" width="120" />
            <el-table-column prop="Address" label="住址" width="300" />
            <el-table-column fixed="right" label="Operations" width="120">
                <template #default="scope">
                    <el-button link type="primary" size="small" @click="handleRowDel(scope.row)" style="color: #F56C6C">
                        删除
                    </el-button>
                    <el-button link type="primary" size="small" @click="handleEdit(scope.row)" >
                        编辑
                    </el-button>
                </template>
            </el-table-column>        
        </el-table>

        <el-pagination 
            background 
            layout="prev, pager, next" 
            style="display: flex; justify-content: center; margin-top: 10px;"
            :total="total" 
            v-model:current-page="curPage"     
            @current-change="handleChangePage"          
        />

        <!-- dialog对话框 -->
        <el-dialog v-model="dialogFormVisible" :title="dialogType === 'add' ? '新增' : '编辑'">
            <el-form :model="tableForm">
                <el-form-item label="姓名" :label-width="60">
                    <el-input v-model="tableForm.Name" autocomplete="off" />
                </el-form-item>
                <el-form-item label="邮箱" :label-width="60">
                    <el-input v-model="tableForm.Email" autocomplete="off" />
                </el-form-item>
                <el-form-item label="电话" :label-width="60">
                    <el-input v-model="tableForm.Phone" autocomplete="off" />
                </el-form-item>
                <el-form-item label="状态" :label-width="60">
                    <el-input v-model="tableForm.State" autocomplete="off" />
                </el-form-item>
                <el-form-item label="地址" :label-width="60">
                    <el-input v-model="tableForm.Address" autocomplete="off" />
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button type="primary" @click="dialogConfirm">
                        确认
                    </el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import request from '../utils/request'

/* 数据 */
const queryInput = ref("")
const multipleSelection = ref([])
let tableData = ref([
    {
        id: "1",
        name: 'Tom1',
        email: '123@qq.com',
        phone: '13855669452',
        state: '在职',
        address: '广东省'
    },
    {
        id: "2",
        name: 'Tom2',
        email: '123@qq.com',
        phone: '13855669452',
        state: '在职',
        address: '广东省'
    },
    {
        id: "3",
        name: 'Tom3',
        email: '123@qq.com',
        phone: '13855669452',
        state: '在职',
        address: '广东省'
    },
    {
        id: "4",
        name: 'Tom4',
        email: '123@qq.com',
        phone: '13855669452',
        state: '在职',
        address: '广东省'
    }
])
const dialogFormVisible = ref(false)
let tableForm = ref({})
let dialogType = ref('add')
let tableDataCopy = Object.assign(tableData)    // 浅拷贝

let total = ref(10)
let curPage = ref(1)

/* 方法 */

const getTableData = async (cur = 1)=> {
    // 第一种请求方式
    let res = await request.get('/list', {
        pageSize: 10,
        pageNum: cur
    })
    // 第二种请求方式
    // let res = await request.get(`/list/?pageSize=10 && pageNum=${cur}`);
    // console.log(res)
    tableData.value = res.list
    total = res.total
    curPage = cur
}
getTableData(1)

/* 请求分页 */
const handleChangePage = (val)=> {
    getTableData(curPage)
}

// 搜索
const handleQueryName = async (val)=> {
    if (val.length > 0) {
        tableData.value = await request.get(`/list/${val}`)
    } else {
        await getTableData(curPage)  
    }
}

//删除一行数据 
const handleRowDel = async ({ID}) => {     
    await request.delete(`/delete/${ID}`)  
    await getTableData(curPage)  
}

// add弹窗
const handleAdd = ()=> {  
    dialogType = 'add'           
    dialogFormVisible.value = true
    tableForm.value = {}
}

// 多选
const handleSelectionChange = (val) => {    
    // multipleSelection.value = val
    // console.log(val);

    multipleSelection.value = []
    val.forEach(item => {
        multipleSelection.value.push(item.ID)
    })
    /* console.log(multipleSelection.value)*/
}

// 删除多选
const handleDelList = ()=> {        
    multipleSelection.value.forEach(ID => {
        handleRowDel({ID})
    })
    multipleSelection.value = []
}

// edit弹窗
const handleEdit = (row)=> {
    dialogFormVisible.value = true
    dialogType = 'edit'
    /* console.log(row)*/
    tableForm.value = {...row}
}

// 确认
const dialogConfirm = async ()=> {  
    dialogFormVisible.value = false

    // 判断新增还是编辑
    if (dialogType === 'add') {
        // 1.拿到数据
        // 2.添加到table
        await request.post("/add",{
            ...tableForm.value
        })

        // 刷新数据
        await getTableData(curPage)

    } else {
        // 修改内容
        await request.put(`/update/${tableForm.value.ID}`,{
            ...tableForm.value
        })
        // console.log(tableForm.value.ID)
        await getTableData(curPage)
    }

    
}

</script>

<style>
.table-box {
    width: 800px;
    margin: 200px auto;
}

.title {
    text-align: center;
}

.query-box {
    display: flex;
    justify-content: space-between;
    margin-bottom: 25px;
}

.queryInput {
    width: 200px;
}
</style>