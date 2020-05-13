<template> 
  <el-card class="form-container" shadow="never">
    <el-steps :active="active" finish-status="success" align-center>
      <el-step title="填写项目信息" />
      <el-step title="填写项目计划" />
      <el-step title="填写完结信息" />
    </el-steps>
    <project-info-detail
      v-show="showStatus[0]"
      v-model="productParam"
      :is-edit="isEdit"
      @nextStep="nextStep">
    </project-info-detail>
    <project-plan-detail
      v-show="showStatus[1]"
      v-model="productParam"
      :is-edit="isEdit"
      @nextStep="nextStep"
      @prevStep="prevStep">
    </project-plan-detail>
    <project-close-detail
      v-show="showStatus[2]"
      v-model="productParam"
      :is-edit="isEdit"
      @prevStep="prevStep"
      @finishCommit="finishCommit">
    </project-close-detail>

  </el-card>
</template>
<script>
  import ProjectInfoDetail from './ProjectInfoDetail';
  import ProjectPlanDetail from './ProjectPlanDetail';
  import ProjectCloseDetail from './ProjectCloseDetail';
  import {createProduct,getProduct,updateProduct} from '@/api/product';
  import {createProjectManage} from '@/api/ProjectManage';

  const defaultProductParam = {
    //项目信息
    projectInfo:{
      projectName: '',//项目名称
      projectCataId:'',//项目分类ID
      projectCataName:'',//项目分类名字
      projectMainMan: '', //项目负责人
      projectMainManId: '', //项目负责人ID
      joinMan: '', //项目成员
      joinManId: '', //项目成员IDS
      important: 0, //项目重要程度
      projectDesc: '', //项目介绍
    },
    //计划信息
    planList: [],
    //项目完结信息
    projectCloseInfo:{
      projectCloseName:'',//完结人
      projectIsClose:'',//是否完结
      closeStatus:'', //完结状态
      finishTime: '', //项目完成时间
    },

  };
  export default {
    name: 'ProjectDetail',
    components: {
      ProjectCloseDetail,
      ProjectPlanDetail,
      ProjectInfoDetail
    },
    props: {
      isEdit: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        active: 0,
        productParam: Object.assign({}, defaultProductParam),
        showStatus: [true, false, false, false]
      }
    },
    created(){
      if(this.isEdit){
        getProduct(this.$route.query.id).then(response=>{
          this.productParam=response.data;
        });
      }
    },
    methods: {
      hideAll() {
        for (let i = 0; i < this.showStatus.length; i++) {
          this.showStatus[i] = false;
        }
      },
      prevStep() {
        if (this.active > 0 && this.active < this.showStatus.length) {
          this.active--;
          this.hideAll();
          this.showStatus[this.active] = true;
        }
      },
      nextStep() {
        if (this.active < this.showStatus.length - 1) {
          this.active++;
          this.hideAll();
          this.showStatus[this.active] = true;
        }
      },
      finishCommit(isEdit) {
        console.log(this.productParam);
        this.$confirm('是否要提交该产品', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          if(isEdit){
            updateProduct(this.$route.query.id,this.productParam).then(response=>{
              this.$message({
                type: 'success',
                message: '提交成功',
                duration:1000
              });
              this.$router.back();
            });
          }else{
            createProjectManage(this.productParam).then(response=>{
              this.$message({
                type: 'success',
                message: '提交成功',
                duration:1000
              });
              location.reload();
            });
          }
        })
      }
    }
  }
</script>
<style>
  .form-container {
    width: 800px;
  }
</style>


