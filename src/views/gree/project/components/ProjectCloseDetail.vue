<template>
  <div style="margin-top: 50px">
    <el-form :model="value" :rules="rules" ref="productCloseForm" label-width="120px" style="width: 600px" size="small">
      <el-form-item label="完结人：" prop="projectCloseName">
        <el-select
          v-model="value.projectCloseInfo.projectCloseName"
          @change="handleBrandChange"
          placeholder="请选择完结人">
          <el-option
            v-for="item in brandOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="是否完结：">
        <el-switch
          v-model="value.projectCloseInfo.projectIsClose"
          :active-value="1"
          :inactive-value="0">
        </el-switch>
      </el-form-item>
      <el-form-item label="完结状态">
        <el-radio-group v-model="value.projectCloseInfo.closeStatus">
          <el-radio label="未完结"/>
          <el-radio label="正常完结" />
          <el-radio label="延期完结" />
          <el-radio label="异常" />
        </el-radio-group>
      </el-form-item>
      <el-form-item label="项目完成时间：">
        <el-date-picker
          class="input-width"
          v-model="value.projectCloseInfo.finishTime"
          value-format="yyyy-MM-dd"
          type="date"
          placeholder="请选择时间">
        </el-date-picker>
      </el-form-item>

      <el-form-item style="text-align: center">
        <el-button size="medium" @click="handlePrev">上一步，填写项目计划</el-button>
        <el-button type="primary" size="medium" @click="handleFinishCommit">完成，提交商品</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import {fetchListWithChildren} from '@/api/productCate'
  import {fetchList as fetchBrandList} from '@/api/brand'

  export default {
    name: "ProjectCloseDetail",
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
        }
      };
    },
    created() {
      // this.getProductCateList();
      this.getBrandList();
    },
    computed:{
      //商品的编号
      // productId(){
      //   return this.value.id;
      // }
    },
    watch: {
      // productId:function(newValue){
      //   if(!this.isEdit)return;
      //   if(this.hasEditCreated)return;
      //   if(newValue===undefined||newValue==null||newValue===0)return;
      //   this.handleEditCreated();
      // },
      selectProductCateValue: function (newValue) {
        if (newValue != null && newValue.length === 2) {
          this.value.projectCloseInfo.productCategoryId = newValue[1];
          this.value.projectCloseInfo.productCategoryName= this.getCateNameById(this.value.projectCloseInfo.productCategoryId);
        } else {
          this.value.projectCloseInfo.productCategoryId = null;
          this.value.projectCloseInfo.productCategoryName=null;
        }
      }
    },
    methods: {
      //处理编辑逻辑
      handleEditCreated(){
        if(this.value.projectCloseInfo.productCategoryId!=null){
          this.selectProductCateValue.push(this.value.cateParentId);
          this.selectProductCateValue.push(this.value.projectCloseInfo.productCategoryId);
        }
        this.hasEditCreated=true;
      },

      getBrandList() {
        fetchBrandList({pageNum: 1, pageSize: 100}).then(response => {
          this.brandOptions = [];
          let brandList = response.data.list;
          for (let i = 0; i < brandList.length; i++) {
            this.brandOptions.push({label: brandList[i].name, value: brandList[i].id});
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
      handlePrev() {
        this.$emit('prevStep')
      },
      handleFinishCommit(){
        this.$emit('finishCommit',this.isEdit);
      },
      handleBrandChange(val) {
        let brandName = '';
        for (let i = 0; i < this.brandOptions.length; i++) {
          if (this.brandOptions[i].value === val) {
            brandName = this.brandOptions[i].label;
            break;
          }
        }
        this.value.brandName = brandName;
      }
    }
  }
</script>

<style scoped>
</style>
