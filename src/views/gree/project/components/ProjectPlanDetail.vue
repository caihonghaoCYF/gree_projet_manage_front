<template>
  <div style="margin-top: 50px">
      <div slot="header" class="clearfix">
        <el-button style="float: right; padding: 3px 0" type="text" @click="addPlan">添加计划</el-button>
      </div>
      <el-form :model="value" ref="productPlanForm" label-width="120px" style="width: 820px" size="small">
        <div v-for="o in planList" :key="o.id" >
          <el-tag>
            计划【{{o.id}}】
          </el-tag>
          <div style="margin-bottom: 10px;">
            <el-form-item label="计划名称：">
              <el-input v-model="o.planName"></el-input>
            </el-form-item>
            <el-form-item label="操作人：" prop="planOperate">
              <el-select
                v-model="o.planOperate"
                @change="handleBrandChange"
                placeholder="请选择成员">
                <el-option
                  v-for="item in brandOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="计划完成时间：">
              <el-date-picker
                class="input-width"
                v-model="o.planFinishTime"
                value-format="yyyy-MM-dd"
                type="date"
                placeholder="请选择时间">
              </el-date-picker>
            </el-form-item>
            <el-form-item label="计划文档：">
              <el-upload
                class="upload-demo"
                drag
                v-model="o.planDoc"
                action="https://jsonplaceholder.typicode.com/posts/"
                multiple>
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                <!--          <div class="el-upload__tip" slot="tip">只能上传jpg/png/pdf/word/excel/md文件，且不超过10m</div>-->
              </el-upload>
            </el-form-item>
            <el-form-item label="计划详情：">
                <tinymce :width="595" :height="300" v-model="o.planDesc"></tinymce>
            </el-form-item>
            <el-divider></el-divider>
          </div>
        </div>

        <el-form-item style="text-align: center">
          <el-button size="medium" @click="handlePrev">上一步，填写项目信息</el-button>
          <el-button type="primary" size="medium" @click="handleNext">下一步，填写完结信息</el-button>
        </el-form-item>
      </el-form>
  </div>
</template>

<script>
  import {fetchList as fetchMemberLevelList} from '@/api/memberLevel'
  import Tinymce from '@/components/Tinymce'
  import {fetchList as fetchBrandList} from '@/api/brand'
  export default {
    name: "ProjectPlanDetail",
    components: {Tinymce},
    props: {
      value: Object,
      isEdit: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        //日期选择器配置
        pickerOptions1: {
          disabledDate(time) {
            return time.getTime() < Date.now();
          }
        },
        brandOptions: [],
        activeHtmlName: 'pc',
        planList:[]
      }
    },
    created() {
      if (this.isEdit) {
        // this.handleEditCreated();
      } else {
        fetchMemberLevelList({defaultStatus: 0}).then(response => {
          let memberPriceList = [];
          for (let i = 0; i < response.data.length; i++) {
            let item = response.data[i];
            memberPriceList.push({memberLevelId: item.id, memberLevelName: item.name})
          }
          this.value.memberPriceList = memberPriceList;
        });
      }
      this.getBrandList();
    },
    computed: {
      //选中的服务保证
      selectProductPics:{
        get:function () {
          let pics=[];
          if(this.value.pic===undefined||this.value.pic==null||this.value.pic===''){
            return pics;
          }
          pics.push(this.value.pic);
          if(this.value.albumPics===undefined||this.value.albumPics==null||this.value.albumPics===''){
            return pics;
          }
          let albumPics = this.value.albumPics.split(',');
          for(let i=0;i<albumPics.length;i++){
            pics.push(albumPics[i]);
          }
          return pics;
        },
        set:function (newValue) {
          if (newValue == null || newValue.length === 0) {
            this.value.pic = null;
            this.value.albumPics = null;
          } else {
            this.value.pic = newValue[0];
            this.value.albumPics = '';
            if (newValue.length > 1) {
              for (let i = 1; i < newValue.length; i++) {
                this.value.albumPics += newValue[i];
                if (i !== newValue.length - 1) {
                  this.value.albumPics += ',';
                }
              }
            }
          }
        }
      },
      selectServiceList: {
        get() {
          let list = [];
          if (this.value.serviceIds === undefined || this.value.serviceIds == null || this.value.serviceIds === '') return list;
          let ids = this.value.serviceIds.split(',');
          for (let i = 0; i < ids.length; i++) {
            list.push(Number(ids[i]));
          }
          return list;
        },
        set(newValue) {
          let serviceIds = '';
          if (newValue != null && newValue.length > 0) {
            for (let i = 0; i < newValue.length; i++) {
              serviceIds += newValue[i] + ',';
            }
            if (serviceIds.endsWith(',')) {
              serviceIds = serviceIds.substr(0, serviceIds.length - 1)
            }
            this.value.serviceIds = serviceIds;
          } else {
            this.value.serviceIds = null;
          }
        }
      }
    },
    methods: {
      getBrandList() {
        fetchBrandList({pageNum: 1, pageSize: 100}).then(response => {
          this.brandOptions = [];
          let brandList = response.data.list;
          for (let i = 0; i < brandList.length; i++) {
            this.brandOptions.push({label: brandList[i].name, value: brandList[i].id});
          }
        });
      },
      addPlan(){
        let length = this.planList.length;
        let planObj = {
          id: length+1,
          planName: "",
          planDesc: "",
          planDoc: "",
          planFinishTime: "",
          planOperate:""
        };
        this.planList.push(planObj);
        console.log(planObj)
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
      },
      handleEditCreated() {
        let ids = this.value.serviceIds.split(',');
        console.log('handleEditCreated', ids);
        for (let i = 0; i < ids.length; i++) {
          this.selectServiceList.push(Number(ids[i]));
        }
      },
      handleRemoveProductLadder(index, row) {
        let productLadderList = this.value.productLadderList;
        if (productLadderList.length === 1) {
          productLadderList.pop();
          productLadderList.push({
            count: 0,
            discount: 0,
            price: 0
          })
        } else {
          productLadderList.splice(index, 1);
        }
      },
      handleAddProductLadder(index, row) {
        let productLadderList = this.value.productLadderList;
        if (productLadderList.length < 3) {
          productLadderList.push({
            count: 0,
            discount: 0,
            price: 0
          })
        } else {
          this.$message({
            message: '最多只能添加三条',
            type: 'warning'
          });
        }
      },
      handleRemoveFullReduction(index, row) {
        let fullReductionList = this.value.productFullReductionList;
        if (fullReductionList.length === 1) {
          fullReductionList.pop();
          fullReductionList.push({
            fullPrice: 0,
            reducePrice: 0
          });
        } else {
          fullReductionList.splice(index, 1);
        }
      },
      handleAddFullReduction(index, row) {
        let fullReductionList = this.value.productFullReductionList;
        if (fullReductionList.length < 3) {
          fullReductionList.push({
            fullPrice: 0,
            reducePrice: 0
          });
        } else {
          this.$message({
            message: '最多只能添加三条',
            type: 'warning'
          });
        }
      },
      handlePrev() {
        this.$emit('prevStep')
      },
      handleNext() {
        this.$emit('nextStep')
      }
    }
  }
</script>

<style scoped>
  .littleMargin {
    margin-top: 10px;
  }
</style>
