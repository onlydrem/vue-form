/**
功能：表单封装
作者：程杰
邮箱：chengjie@cebc.cn
创建时间：2021/1/14 11:20
**/
<template>
  <div class="voucher-form">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span class="card-name">凭证信息</span>
      </div>
      <div class="form-item-list">
        <el-form ref="formBasic" :model="initFormItem.basicForm" key="basic" >
         <el-form-item v-for="basic in initFormItem.basic_form"
                       :key="basic.prop"
                       :label-width="basic.labelWidth"
                       :rules="basic.rules"
                       :label="basic.label"
                       :prop="basic.prop">
           <template v-if="basic.type === 'voucherInput'">
             <el-input
                       onkeyup="value=value.replace(/[^\d{1,}\.\d{1,}|\d{1,}]/g,'')"
                       :style="{width: basic.width}"
                       v-model.trim="initFormItem.basicForm[basic.prop]"
                       :placeholder="basic.placeholder"
                       autocomplete="off"><i slot="suffix"
                                             style="font-style:normal;margin-right: 10px;">元</i></el-input>
             <span style="line-height: 14px; display: inline-block; margin-left: 8px">
                可授信额度： {{availableCreditLimit ? price(availableCreditLimit): price(0)}} </span>
             <span class="voucher-amount-message" v-if="voucherError">{{voucherError}}</span>
           </template>
            <template v-if="basic.type === 'Radio'">
              <div class="financing-class">
                <el-radio-group v-model="initFormItem.basicForm[basic.prop]">
                  <el-radio border  v-for="bank in basic.options" :key="bank.id"
                            :label="bank.bankOrgId" @change="getRadioId(bank.bankOrgId)">
                    <div class="bank-list">
                      <div class="bank-icon">
                        <div class="bank-img">
                          <img :src="bank.url ? bank.url : default_bank" alt="">
                        </div>
                      </div>
                      <div class="bank-name-box">
                        <div class="bank-name">
                          <div class="bank-name-list">{{
                            bank.bankOrgName
                            }}</div>
                          <span>授信可用额度：<span class="money">{{price(bank.availableCreditLimit) }}
                                                    </span>元</span>
                        </div>
                      </div>
                    </div>
                  </el-radio>
                </el-radio-group>
              </div>
            </template>
           <el-select  v-if="basic.type === 'Account'" v-model="initFormItem.basicForm[basic.prop]"
                      :placeholder="basic.placeholder"
                       :style="{width: basic.width}">
             <el-option v-for="account in basic.options"
                        :key="account.id"
                        :label='`【${account.accountSonNo}】  【${account.bankName}】`'
                        :value="account.orgId">
             </el-option>
           </el-select>
           <template v-if="basic.type === 'Frezzy'">
             <FuzzyQuery
               :queryType="initFormItem.queryType"
               @invitionBusiness="invitionBusiness"
               @addBusiness="addBusiness"
               @getObjById="getObjById" />
           </template>
           <el-date-picker v-if="basic.type === 'promiseDate'" v-model="initFormItem.basicForm[basic.prop]"
                           :placeholder="basic.placeholder"
                           :style="{width: basic.width}"
                           type="date"
                           value-format="yyyy-MM-dd"
                           format="yyyy-MM-dd"
                           :picker-options="expireTimeOption">
           </el-date-picker>
           <el-input v-if="basic.type === 'TextArea'" v-model="initFormItem.basicForm[basic.prop]"
                     :placeholder="basic.placeholder"
                     :style="{width: basic.width}"
                     type="textarea"
                     :rows="3" />
          <slot v-if="basic.type==='slot'" :name="basic.slotName"></slot>
         </el-form-item>
        </el-form>
      </div>
    </el-card>

    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span class="card-name">项目信息</span>
      </div>
      <div class="form-item-list">
        <el-form ref="formProject" :model="initFormItem.projectForm" key="basic">
          <el-form-item v-for="project in initFormItem.project_form"
                        :key="project.prop"
                        :label="project.label"
                        :prop="project.prop"
                        :rules="project.rules"
                        :label-width="project.labelWidth">
            <el-input v-if="project.type === 'projectInput'"
                      v-model.trim="initFormItem.projectForm[project.prop]"
                      type="text"
                      :placeholder="project.placeholder"
                      :style="{width: project.width}">
            </el-input>
            <template v-if="project.type === 'projectUpload'">
              <el-row>
                <el-col :span="17">
                  <el-button type="primary"
                             class="icon-button"
                             @click="project.handler && project.handler()">
                    <svg-icon icon-class="base-add"
                              style="display: inline-block;"></svg-icon>
                    {{project.btnName}}
                  </el-button>
                </el-col>
                <el-col :span="7"
                        style="text-align: right">
                  <span>{{project.amountName}}：<span class="money">{{price(project.transactionTotal)}}</span>元 </span>
                </el-col>
              </el-row>
            </template>
            <template v-if="project.type === 'contractTable'">
              <el-table :data="project.tableContractData"
                        border>
                <el-table-column type="index"
                                 label="序号"
                                 width="50">
                </el-table-column>
                <el-table-column label="合同金额(元)"
                                 min-width="20%">
                  <template slot-scope="scope">
                    <div>
                      {{ price( scope.row.contractAmount) }}
                    </div>
                  </template>
                </el-table-column>
                <el-table-column prop="contractName"
                                 label="合同名称"
                                 min-width="20%">
                </el-table-column>
                <el-table-column prop="contractNumber"
                                 label="合同编号"
                                 min-width="20%">
                </el-table-column>
                <el-table-column prop="signatory1"
                                 label="合同签署方一"
                                 min-width="20%">
                </el-table-column>
                <el-table-column prop="signatory2"
                                 label="合同签署方二"
                                 min-width="20%"></el-table-column>
                <el-table-column label="操作"
                                 width="160">
                  <template slot-scope="scope">
                    <el-button type="text"
                               size="small"
                               @click="downloadContract(scope.$index, scope.row)">
                      下载
                    </el-button>
                    <el-button type="text"
                               size="small"
                               @click="project.handler && project.handler(scope.$index, scope.row)">
                      编辑
                    </el-button>
                    <el-button type="text"
                               size="small"
                               @click.native.prevent="delContract(scope.$index, scope.row)">
                      删除
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
            </template>
            <template v-if="project.type === 'InvoiceTable'">
              <el-table :data="project.tableInvoiceData"
                        border
                        style="width: 100%">
                <el-table-column type="index"
                                 label="序号"
                                 width="50">
                </el-table-column>
                <el-table-column prop="invoiceCode"
                                 label="发票代码"
                                 min-width="20%">
                </el-table-column>
                <el-table-column prop="invoiceNumber"
                                 label="发票号码"
                                 min-width="20%">
                </el-table-column>
                <el-table-column label="发票金额(含税)(元)"
                                 min-width="20%">
                  <template slot-scope="scope">
                    <div>
                      {{ price( scope.row.invoiceTotalAmount) }}
                    </div>
                  </template>
                </el-table-column>
                <el-table-column label="发票金额(不含税)(元)"
                                 min-width="20%">
                  <template slot-scope="scope">
                    <div>
                      {{ price( scope.row.invoiceAmount) }}
                    </div>
                  </template>
                </el-table-column>
                <el-table-column label="开票日期"
                                 min-width="20%">
                  <template slot-scope="scope">
                    <div>
                      {{   parseTime( scope.row.billingDate, '{y}-{m}-{d}') }}
                    </div>
                  </template>
                </el-table-column>
                <el-table-column label="操作"
                                 width="160">
                  <template slot-scope="scope">
                    <el-button type="text"
                               size="small"
                               @click="downloadInvoice(scope.$index, scope.row)">
                      下载
                    </el-button>
                    <el-button type="text"
                               size="small"

                      @click="project.handler && project.handler(scope.$index, scope.row)">
                      编辑
                    </el-button>
                    <el-button type="text"
                               size="small"
                               @click.native.prevent="delInvoice(scope.$index, scope.row)">
                      删除
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
            </template>
            <el-upload v-if="project.type === 'otherFileUpload'" class="upload-demo"
                       ref="upload"
                       action=""
                       name="file"
                       accept=".jpg,.jpeg,.png,.pdf,.rar ,.zip,.doc,.docx"
                       :on-preview="handlePreview"
                       :file-list="fileList"
                       :http-request="addOtherFiles"
                       :before-remove="(response,file,fileList)=>beforeRemove(response,file,fileList,)"
                       :before-upload="(response, file,fileList )=>beforeUpload(response,file, fileList)"
                       :on-success="(response,file,fileList)=>handleAvatarSuccess1(response,file,fileList)"
                       :on-remove="(response,file,fileList)=>handleRemove(response,file,fileList)">
              <el-button class="icon-button"
                         type="primary"
                         slot="trigger">
                <svg-icon icon-class="base-upload"
                          style="margin-right: 4px"></svg-icon>
                {{project.btnName}}
              </el-button>
            <span slot="tip"
                   class="el-upload__tip upload_tips">
                支持类型：jpg、jpeg、png、pdf、rar 、zip 、doc 、docx ；文件大小：30M内
             </span>
            </el-upload>
            <template v-if="project.type === 'passwordInput'">
              <el-input v-model.trim="initFormItem.projectForm[project.prop]"  :placeholder="project.placeholder" :style="{width: project.width}"
                        type="password"
                        >
              </el-input>
              <span style="display: inline-block;margin-left: 20px;color:#00B39B;cursor:pointer; "
                    @click="project.handler&& project.handler()">忘记密码？</span>

            </template>
          </el-form-item>
        </el-form>
      </div>
    </el-card>
  </div>
</template>

<script>
  import FuzzyQuery from '@p/voucherComponent/fuzzy-query'
  import { FileMixin } from '@p/common-modules/cebc-voucher/voucher-info/file-data/index.js'
  import { businessMixin } from "@p/common-modules/business-management/index.js";
  import { commonMixin } from '@p/common-modules/cebc-voucher/voucher-info/mixin/global.js'
    export default {
        name: 'form-component',
        props: {
          setFormItem: {
            type: Object,
            default: ()=> ({})
          }
        },
      mixins: [
        businessMixin,
        FileMixin,
        commonMixin
      ],
        components: {
          FuzzyQuery
        },
        data() {
            return {
              availableCreditLimit: '',
              voucherError: '',
              receiptDataDetail: {},
              initFormItem: {
                queryType: '',
                voucherName: '',
                projectName: '',
                basic_form: [],
                project_form: [],
                basicForm: {},
                projectForm:{}
              },
              require_message: {
                "Input": "请输入",
                "Account": "请输入",
                "Frezzy": "请输入",
                "voucherInput": "请输入",
                "projectInput": "请输入",
                "passwordInput": "请输入",
                "projectUpload": "请上传",
                "promiseDate": "请选择",
                "InputNumber": "请输入",
                "Radio": "请选择",
                "Select": "请选择",
                "Disabled": "请选择",
                "Upload": "请上传"
              },
            }
        },
        computed: {
          ownFinancingCreditArr () {
            return this.$store.state.metadata.ownFinancingCreditArr;
          },
          defaultBankFlagId () {
            return this.$store.state.metadata.defaultBankFlagId;
          },
        },
        watch: {
          setFormItem: {
            handler(newVal) {
              this.initFormData()
            },
            immediate: true,
            deep: true
          },
          availableCreditLimit: {
            handler(newVal) {
              if(this.initFormItem.basicForm.voucherAmount * 100 > newVal) {
                this.voucherError = '凭证金额不能大于授信可用金额'
              }
              else if(this.initFormItem.basicForm.voucherAmount != '' &&
                      this.initFormItem.basicForm.voucherAmount * 100 < 100) {
                        this.voucherError = '请输入不小于1的凭证金额'
              }
              else{
                this.voucherError = ''
              }
            },
            immediate: true,
            deep: true
          },
          'initFormItem.basicForm.voucherAmount': {
            handler(newVal) {
              if(newVal != '' && newVal * 100 < 100) {
                this.voucherError = '请输入不小于1的凭证金额'
              }
              else if(newVal * 100 > this.availableCreditLimit) {
                this.voucherError = '凭证金额不能大于授信可用金额'
              }
             else {
                this.voucherError = ''
              }
            },
            immediate: true,
            deep: true
          }
        },
        methods: {
          // 初始化表单数据
          initFormData() {
            for(let key in this.setFormItem) {
              if(Object.keys(this.initFormItem).includes(key)) {
                this.initFormItem[key] = this.setFormItem[key]
              }
            }
            this.initFormItem.basicForm.paymentBankId = this.defaultBankFlagId
            this.initFormItem.basic_form.forEach(item=> {
              if(item.required) { this.rules(item); }
              // 自定义检验规则
              if(item.validator) { item.rules = item.validator; }
            })
            this.initFormItem.project_form.forEach(item=> {
              if(item.required) { this.rules(item); }
              // 自定义检验规则
              if(item.validator) { item.rules = item.validator; }
            })
          },
          // 获取选中的融资行的可用额度
          async getRadioId (id) {
            if (id) {
              const getAvailableCreditLimit = this.ownFinancingCreditArr.filter(item => item.bankOrgId === id)
              this.availableCreditLimit = getAvailableCreditLimit[0].availableCreditLimit
            }
          },
          // 获取模糊查询的详情
          getObjById (data) {
            if (JSON.stringify(data) != "{}") {
              this.ruleFormBasic.receiptName = data.orgName
              this.$refs.receiptName.clearValidate();
              this.receiptDataDetail = data
            } else {
              this.receiptDataDetail = {}
            }
          },
          rules(item){
            const requiredRules = [{ required: true, message: item.required_msg || `${this.require_message[item.type]}${item.label}`, trigger: item.trigger }];
            // 其他的 rules 规则
            if(item.rules && item.rules.length > 0) {
              item.rules = requiredRules.concat(item.rules);
            }else{
              item.rules = requiredRules;
            }
          }
        }
    }
</script>

<style lang="scss" scoped>
  @import "~@as/styles/mixin.scss";
  @import "~@as/styles/variables.scss";
  @import "~@as/styles/main.scss";
  .voucher-form{
    margin-bottom: 32px;
    .clearfix:before,
    .clearfix:after {
      display: table;
      content: "";
    }
    .el-card{
      border: 0 !important;
      box-shadow: 0 0px 0px #ccc!important;
    }
    .clearfix:after {
      clear: both
    }
    .card-name{
      font-size: 16px;
      font-weight: 500;
      color: #333333;
      line-height: 24px;
    }
    .form-item-list{
      width: 76%;
      height: auto;
      margin: 16px auto;
      ::v-deep .el-form-item {
        margin-bottom: 24px !important;
      }
      .voucher-amount-message{
        color: red;
        line-height: 14px;
        display: inline-block;
        margin-left: 8px
      }
      .financing-class {
        ::v-deep .el-radio__inner {
          display: none !important;
        }

        .is-checked {
          background: rgba(0, 179, 155, 0.09) !important;
        }

        ::v-deep .el-radio__label {
          padding: 0 !important;
        }

        ::v-deep .el-radio__input {
          display: none !important;
        }

        ::v-deep .el-upload-list {
          display: block !important;
        }

        ::v-deep .el-radio {
          height: auto !important;
          display: inline-block;
          margin-left: 0 !important;
          margin-right: 8px !important;
          // background: rgba(0, 179, 155, 0.09) !important;
          padding: 0 !important;
          // margin: 0 !important;
          margin-bottom: 8px;
          // margin-right: 8px !important;
        }
      }
      .bank-list {
        @include clearfix();
        @include flex();
        width: 320px;
        height: 98px;
        border: none !important;
        background: transparent !important;
        border-radius: 4px;
        overflow: hidden;
        .bank-icon {
          width: 80px;
          height: 100%;
          box-sizing: border-box;
          @include flex();
          align-items: center;
          justify-content: center;
          .bank-img {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            img {
              display: block;
              width: 100%;
              height: 100%;
            }
          }
        }
        .bank-name-box {
          @include flex();
          align-items: center;
          justify-content: center;
        }
        .bank-name {
          width: 230px;
          box-sizing: border-box;
          white-space: normal;
          .money {
            font-size: 14px;
            font-weight: 400;
            color: #fa6400;
            line-height: 22px;
          }
          .bank-name-list {
            font-size: 16px;
            font-weight: 500;
            color: rgba(51, 51, 51, 1);
            line-height: 20px;
            font-weight: 600;
            width: 100%;
            height: auto;
            white-space: normal;
            word-break: break-all;
            word-wrap: break-word;
          }
          span {
            font-size: 14px;
            font-weight: 400;
            color: rgba(0, 0, 0, 0.25);
            line-height: 22px;
          }
        }
        .el-table_line {
          height: 1px;
          background: rgba(238, 238, 238, 1);
          margin-top: 24px;
          margin-bottom: 16px;
          width: 100%;
          margin: auto;
        }
        .el-upload__tip {
          margin-left: 18px;
          font-size: 14px;
          font-weight: 400;
          color: rgba(153, 153, 153, 1);
          line-height: 22px;
        }
        .el-upload-list {
          display: block !important;
          align-items: center;
          margin-right: 20px;
        }
        .el-upload-list__item:first-child {
          margin-right: 20px;
        }
      }
      .upload_tips {
        margin-left: 16px;
      }
    }
  }
</style>
