<template>
  <div class="hello">
    <el-button style="float:left;" size="small" type="primary" @click="handleChooseData">{{title}}</el-button>
    <el-dialog title="客户选择" :visible.sync="dialogVisible" width="80%">
      <el-button style="float:left;" size="small" type="primary" @click="handleClearData">清空所有</el-button>
      <el-button style="float:left;" size="small" type="primary" @click="handleAddData">添加选中的</el-button>
      <el-table :data="tableData" ref="table" @selection-change="handleSelectionChange">
        <el-table-column type="selection"></el-table-column>
        <el-table-column prop="personName" label="客户名称"></el-table-column>
        <el-table-column prop="telphone" label="手机号"></el-table-column>
        <el-table-column prop="idNo" label="身份证号"></el-table-column>
        <el-table-column prop="linkType" label="客户身份"></el-table-column>
      </el-table>
      <el-pagination
        :page-size="pagination.pageSize"
        @current-change="currentChange"
        :current-page="pagination.pageNumber"
        :page-sizes="pagination.pageSizes"
        :total="pagination.totalRows"
        :layout="pagination.layout"
        @size-change="sizeChange"
      ></el-pagination>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "ChooseDialogTalbe",
  props: ["title", "checkedData"],
  data() {
    return {
      dialogVisible: false,
      multipleSelectionAll: [], // 所有选中的数据包含跨页数据
      multipleSelection: [], // 当前页选中的数据
      idKey: "personId", // 标识列表数据中每一行的唯一键的名称(需要按自己的数据改一下)
      tableData: [], // 表格数据
      pagination: {
        totalRows: 0, //总条数
        pageSize: 10, //每页显示条数
        pageSizes: [10, 20, 50],
        pageNumber: 1,
        layout: "total, sizes, prev, pager, next, jumper"
      }
    };
  },
  methods: {
    handleChooseData() {
      this.pagination.pageNumber = 1;
      this.queryData();
      this.dialogVisible = true;
    this.$nextTick(function() {
      // 初始化渲染
      this.pagination.pageNumber = 1;
      this.queryData();
    });
    },
    handleClearData() {
      this.multipleSelectionAll = [];
      this.$emit("handleChooseData", []);
      this.dialogVisible = false;
    },
    handleAddData() {
      this.changePageCoreRecordData();
      if (this.multipleSelectionAll.length <= 0) {
        this.$message({ message: "无勾选数据！", type: "warning" });
        return;
      }
      this.$emit("handleChooseData", this.multipleSelectionAll);
      this.dialogVisible = false;
    },
    // 设置选中的方法
    setSelectRow() {
      if (!this.multipleSelectionAll || this.multipleSelectionAll.length <= 0) {
        return;
      }
      // 标识当前行的唯一键的名称
      let idKey = this.idKey;
      let selectAllIds = [];
      let that = this;
      this.multipleSelectionAll.forEach(row => {
        selectAllIds.push(row[idKey]);
      });
      this.$refs.table.clearSelection(); 
      for (var i = 0; i < this.tableData.length; i++) {
        if (selectAllIds.indexOf(this.tableData[i][idKey]) >= 0) {
          // 设置选中，记住table组件需要使用ref="table"
          this.$refs.table.toggleRowSelection(this.tableData[i], true);
        }
      }
    },
    // 记忆选择核心方法
    changePageCoreRecordData() {
      // 标识当前行的唯一键的名称
      let idKey = this.idKey;
      let that = this;
      // 如果总记忆中还没有选择的数据，那么就直接取当前页选中的数据，不需要后面一系列计算
      if (this.multipleSelectionAll.length <= 0) {
        this.multipleSelectionAll = this.multipleSelection;
        return;
      }
      // 总选择里面的key集合
      let selectAllIds = [];
      this.multipleSelectionAll.forEach(row => {
        selectAllIds.push(row[idKey]);
      });
      let selectIds = [];
      // 获取当前页选中的id
      this.multipleSelection.forEach(row => {
        selectIds.push(row[idKey]);
        // 如果总选择里面不包含当前页选中的数据，那么就加入到总选择集合里
        if (selectAllIds.indexOf(row[idKey]) < 0) {
          that.multipleSelectionAll.push(row);
        }
      });
      let noSelectIds = [];
      // 得到当前页没有选中的id
      this.tableData.forEach(row => {
        if (selectIds.indexOf(row[idKey]) < 0) {
          noSelectIds.push(row[idKey]);
        }
      });
      noSelectIds.forEach(id => {
        if (selectAllIds.indexOf(id) >= 0) {
          for (let i = 0; i < that.multipleSelectionAll.length; i++) {
            if (that.multipleSelectionAll[i][idKey] == id) {
              // 如果总选择中有未被选中的，那么就删除这条
              that.multipleSelectionAll.splice(i, 1);
              break;
            }
          }
        }
      });
    },
    currentChange(val) {
      // 改变页的时候调用一次
      this.changePageCoreRecordData();
      this.pagination.pageNumber = val;
      this.queryData();
    },
    sizeChange(val) {
      // 改变每页显示条数的时候调用一次
      this.changePageCoreRecordData();
      this.pagination.pageSize = val;
      this.queryData();
    },
    handleSelectionChange(val) {
      // table组件选中事件,记得加上@selection-change="handleSelectionChange"
      this.multipleSelection = val;
    },
    queryData() {
      // start-----模拟动态分页
      this.tableData = [];
      this.pagination.totalRows = 100;
      let i = (this.pagination.pageNumber - 1) * this.pagination.pageSize + 1;
      let max = this.pagination.pageNumber * this.pagination.pageSize;
      for (; i <= max; i++) {
        this.tableData.push({
          personId: i,
          personName: "小明同志" + i,
          telphone: "137000000" + i,
          idNo: i + "10212000000000011",
          linkType: "业主"
        });
      }
      // end------模拟动态分页
      setTimeout(() => {
        this.setSelectRow();
      }, 20);
    }, // 得到选中的所有数据
    getAllSelectionData() {
      // 再执行一次记忆勾选数据匹配，目的是为了在当前页操作勾选后直接获取选中数据
      this.changePageCoreRecordData();
      console.log(this.multipleSelectionAll);
    }
  },
  mounted: function() {
  },
  watch: {
    checkedData: {
      handler(val) {
        // 转换一下目的是为了不被同步
        this.multipleSelectionAll = JSON.parse(JSON.stringify(val));
      },
      immediate: true,
      deep: true
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.el-pagination {
  background: none;
  text-align: right;
  margin-top: 10px;
}
</style>