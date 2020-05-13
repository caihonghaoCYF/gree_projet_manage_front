<template>
  <div style="margin-top: 50px">
    <el-form :model="value.projectInfo" :rules="rules" ref="productInfoForm" label-width="120px" style="width: 600px" size="small">
      <el-form-item label="项目名称：" prop="projectName">
        <el-input v-model="value.projectInfo.projectName" />
      </el-form-item>
      <el-form-item label="项目类别：" prop="projectCataId">
        <el-cascader
          v-model="selectProductCateValue"
          :options="productCateOptions">
        </el-cascader>
      </el-form-item>
      <el-form-item label="项目负责人：" prop="projectMainMan">
        <el-select
          v-model="value.projectInfo.projectMainManId"
          @change="handleBrandChange"
          placeholder="请选择项目负责人">
          <el-option
            v-for="item in brandOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="项目成员：" prop="joinManId">
        <el-select
          multiple
          v-model="value.projectInfo.joinManId"
          @change="handleBrandChange1"
          placeholder="请选择成员">
          <el-option
            v-for="item in brandOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="项目重要程度：">
        <el-rate
          v-model="value.projectInfo.important">
        </el-rate>
      </el-form-item>
      <el-form-item label="项目介绍：">
        <el-input :autoSize="true" v-model="value.projectInfo.projectDesc" type="textarea" placeholder="请输入内容" />
      </el-form-item>

      <el-form-item style="text-align: center">
        <el-button type="primary" size="medium" @click="handleNext('productInfoForm')">下一步，填写项目计划</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import {fetchListWithChildren} from '@/api/productCate'
  import {fetchList as fetchBrandList} from '@/api/brand'
  import {listByDepartment} from '@/api/login'

  export default {
    name: "ProjectInfoDetail",
    props: {
      value: Object,
      isEdit: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        hasEditCreated:false,
        //选中商品分类的值
        selectProductCateValue: [],
        productCateOptions: [],
        brandOptions: [],
        rules: {
          projectName: [
            {required: true, message: '请输入项目名称', trigger: 'blur'},
            {min: 2, max: 140, message: '长度在 2 到 140 个字符', trigger: 'blur'}
          ],
          projectMainMan: [{required: true, message: '请输入项目负责人', trigger: 'blur'}],
          projectCataId: [{required: true, message: '请选择商品分类', trigger: 'blur'}],
          joinMan: [{required: true, message: '请选择项目成员', trigger: 'blur'}],
          projectDesc: [{required: true, message: '请输入项目介绍', trigger: 'blur'}],
          requiredProp: [{required: true, message: '该项为必填项', trigger: 'blur'}]
        }
      };
    },
    created() {
      this.getProductCateList();
      this.getBrandList();
    },
    computed:{
      //商品的编号
      // productId(){
      //   return this.value.id;
      // }
    },
    watch: {

      selectProductCateValue: function (newValue) {
        if (newValue != null && newValue.length === 2) {
          this.value.projectInfo.projectCataId = newValue[1];
          this.value.projectInfo.projectCataName= this.getCateNameById(this.value.projectInfo.projectCataId);
        } else {
          this.value.projectInfo.projectCataId = null;
          this.value.projectInfo.projectCataName=null;
        }
      }
    },
    methods: {
      //处理编辑逻辑
      handleEditCreated(){
        if(this.value.projectInfo.projectCataId!=null){
          this.selectProductCateValue.push(this.value.projectInfo.cateParentId);
          this.selectProductCateValue.push(this.value.projectInfo.projectCataId);
        }
        this.hasEditCreated=true;
      },
      getProductCateList() {
        fetchListWithChildren().then(response => {
          let list = response.data;
          this.productCateOptions = [];
          for (let i = 0; i < list.length; i++) {
            let children = [];
            if (list[i].children != null && list[i].children.length > 0) {
              for (let j = 0; j < list[i].children.length; j++) {
                children.push({label: list[i].children[j].name, value: list[i].children[j].id});
              }
            }
            this.productCateOptions.push({label: list[i].name, value: list[i].id, children: children});
          }
        });
      },
      getBrandList() {
        listByDepartment().then(response => {
          this.brandOptions = [];
          let brandList = response.data;
          for (let i = 0; i < brandList.length; i++) {
            this.brandOptions.push({label: brandList[i].username, value: brandList[i].id});
          }
        });
      },
      getCateNameById(id){
        let name=null;
        for(let i=0;i<this.productCateOptions.length;i++){
          for(let j=0;j<this.productCateOptions[i].children.length;j++){
            if(this.productCateOptions[i].children[j].value===id){
              name=this.productCateOptions[i].children[j].label;
              return name;
            }
          }
        }
        return name;
      },
      handleNext(formName){
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$emit('nextStep');
          } else {
            this.$message({
              message: '验证失败',
              type: 'error',
              duration:1000
            });
            return false;
          }
        });
      },
      handleBrandChange(val) {
        let brandName = '';
        for (let i = 0; i < this.brandOptions.length; i++) {
          if (this.brandOptions[i].value === val) {
            brandName = this.brandOptions[i].label;
            break;
          }
        }
        this.value.projectInfo.projectMainMan = brandName;
      },
      handleBrandChange1(val) {
        let arrayList = [];
        for(let i = 0; i < this.brandOptions.length; i++){
          for(let j = 0; j < val.length; j++){
            if (this.brandOptions[i].value === val[j]){
              arrayList.push(this.brandOptions[i].label);
            }
          }
        }
        console.log("成员", arrayList)
        this.value.projectInfo.joinMan = arrayList;
      }
    }
  }
</script>

<style scoped>
</style>
