<template>
  <div style="padding: 20px;">
    <!--  //查询-->
    <el-collapse v-model='activeCollapse'>
      <el-collapse-item title='查询条件' name='search'>
        <el-form ref='searchCondition' :model='filter' label-width='100px' label-position=‘left’>
          <el-row>
            <el-col :span='6'>
              <el-form-item label='SDK版本号' prop='versionName'>
                <el-input v-model='filter.versionName' :clearable='true'></el-input>
              </el-form-item>
            </el-col>
            <el-col :span='6' style="marginLeft:20px;">
              <el-button @click='handleSearch' type='primary'>查询</el-button>
              <el-button @click='resetForm("searchCondition")'>重置</el-button>
            </el-col>
<!--            <el-col :span='2' style='margin: 0px 0 10px 0;' type='flex' justify='end'>-->
<!--              <el-button type="primary" @click="dialogFormVisible = true">关联接口</el-button>-->
<!--            </el-col>-->
          </el-row>
        </el-form>
      </el-collapse-item>
    </el-collapse>
    <!--列表-->
    <div>

      <div style='margin-bottom: 20px'>
        <el-table :data='versionList' width='100%' border>
          <el-table-column type='index' width='65' label="序号"></el-table-column>
          <el-table-column label="SDK版本信息">
          <el-table-column label='迭代版本' prop='data.sdk_version' width='150'></el-table-column>
          <el-table-column  label='版本号' prop='data.sdk_iteration_version' width='150'></el-table-column>
            <el-table-column label='所属平台' prop='data.platform' width='150'></el-table-column>
          </el-table-column>
          <el-table-column label='关联接口' width='150'>
            <template slot-scope="scope">
              <div v-for="item in scope.row.interfaceList" >{{item.interface_name}}</div>
            </template>
          </el-table-column>
          <el-table-column label='操作' prop='operate'>
            <template scope="scope">
              <el-button @click="deleteIterationVersion(scope)" type="primary" size="primary">删除关联</el-button>
              <el-button type="primary" @click="updateIterationVersion(scope), dialogupdataVisible = true">修改</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <el-row type='flex' justify='end'>
        <el-pagination
          @size-change='handlePageSizeChange'
          @current-change='handleCurrentChange'
          :current-page='pagination.current'
          :page-sizes='[10, 20, 50, 100]'
          :page-size='pagination.pageSize'
          :total='pagination.total'
          layout='prev,pager,next,jumper,total,sizes'
        ></el-pagination>
      </el-row>
    </div>

    <!--    新增-->
<!--    <el-dialog title="关联接口" :visible.sync="dialogFormVisible">-->
<!--      <el-form :model="form">-->
<!--        <el-form-item label="迭代版本" :label-width="formLabelWidth">-->
<!--          <el-select v-model="value" placeholder="请选择">-->
<!--            <el-option-->
<!--              v-for="item in options1"-->
<!--              :key="item.value1"-->
<!--              :label="item.label1"-->
<!--              :value="item.value1">-->
<!--            </el-option>-->
<!--          </el-select>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="版本号" :label-width="formLabelWidth">-->
<!--          <el-select v-model="value" placeholder="请选择">-->
<!--            <el-option-->
<!--              v-for="item in options2"-->
<!--              :key="item.value2"-->
<!--              :label="item.label1"-->
<!--              :value="item.value2">-->
<!--            </el-option>-->
<!--          </el-select>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="所属平台" :label-width="formLabelWidth">-->
<!--          <el-select v-model="form.platform" placeholder="请选择所属平台">-->
<!--            <el-option label="Android" value="android"></el-option>-->
<!--            <el-option label="IOS" value="ios"></el-option>-->
<!--          </el-select>-->
<!--        </el-form-item>-->


<!--        <div style="margin: 15px 0;"></div>-->
<!--        <el-checkbox-group v-model="checkList">-->
<!--&lt;!&ndash;          <el-checkbox :indeterminate="isIndeterminate" :>全选</el-checkbox>&ndash;&gt;-->
<!--          <el-checkbox v-for='item in checkList' :label="item.checkName"></el-checkbox>-->
<!--&lt;!&ndash;          <el-checkbox label="接口名称 B"></el-checkbox>&ndash;&gt;-->
<!--&lt;!&ndash;          <el-checkbox label="接口名称 C"></el-checkbox>&ndash;&gt;-->
<!--&lt;!&ndash;          <el-checkbox label="接口名称 D" ></el-checkbox>&ndash;&gt;-->
<!--&lt;!&ndash;          <el-checkbox label="接口名称 E" ></el-checkbox>&ndash;&gt;-->
<!--        </el-checkbox-group>-->

<!--      </el-form>-->
<!--      <div slot="footer" class="dialog-footer">-->
<!--        <el-button @click="dialogFormVisible = false">取 消</el-button>-->
<!--        <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>-->
<!--      </div>-->
<!--    </el-dialog>-->
    <!--    修改-->
    <el-dialog title="修改" :visible.sync="dialogupdataVisible">
      <el-form :model="updateform">
        <el-form-item label="迭代版本" :label-width="formLabelWidth">
          {{updateform.sdk_version}}
        </el-form-item>
        <el-form-item label="版本号" :label-width="formLabelWidth">
          {{updateform.sdk_iteration_version}}
        </el-form-item>
        <el-form-item label="所属平台" :label-width="formLabelWidth">
          {{updateform.platform}}
        </el-form-item>

<!--        <el-checkbox :indeterminate="isIndeterminate" >全选</el-checkbox>-->
        <div style="margin: 15px 0;"></div>
        <el-checkbox-group v-model="checkList" v-for='item in updateform.interfaceList' v-if="item.interface_name">
<!--          <el-checkbox :indeterminate="isIndeterminate" :>全选</el-checkbox>-->
          <el-checkbox :label="item.interface_name" :checked="item.checkStatus" ></el-checkbox>
        </el-checkbox-group>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogupdataVisible = false">取 消</el-button>
        <el-button type="primary" @click="postInfo(updateform), dialogupdataVisible = false">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script type="text/javascript">

    import data from './data'
    import methods from './method'
    export default{
        data(){
            return data.init();
        },
        methods:methods,
        mounted(){
            this.getTableData();
            // this.getTableDataFromSDKInterface();
        }
    }
</script>

<style type="text/css">

</style>
