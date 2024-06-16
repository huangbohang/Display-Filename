<script>
  import { bitable,FieldType } from '@lark-base-open/js-sdk';
  import { ref, onMounted } from 'vue';
  import { i18n } from '@/locales/i18n.js'
  import {
    ElButton,
    ElForm,
    ElFormItem,
    ElSelect,
    ElOption,
  } from 'element-plus';

  export default {
    components: {
      ElButton,
      ElForm,
      ElFormItem,
      ElSelect,
      ElOption,
    },
    setup() {
      const formData = ref({ table: '' });
      const tableMetaList = ref([]);
      const options = ref([]);
      const selected_options = ref([]);
      const addname = ref(i18n.global.t('wjm') );

      // const addRecord = async () => {
      //   const tableId = formData.value.table;
      //   if (tableId) {
      //     const table = await bitable.base.getTableById(tableId);
      //     table.addRecord({ fields: {} });
      //   }
      // };
      const reloadOptions  = async (e) => {
        console.log('change')
        const table = await bitable.base.getTableById(formData.value.table);
        const attachmentFieldList = await table.getFieldListByType(FieldType.Attachment);
        const recordList = await table.getRecordList();
        options.value = [];
        for(var i=0;i<attachmentFieldList.length;i++){
          const af = attachmentFieldList[i]
          const data = await table.getFieldMetaById(af.id)
          options.value.push({value:data.name,label:data.name})
        }
        selected_options.value = []
      };
      const exportall = async () => {
        if(selected_options.value.length==0 || addname.value.length==0){
          await bitable.ui.showToast({
            toastType: "error",
            message: i18n.global.t('qwz')
          })
          return 
        }
        const table = await bitable.base.getTableById(formData.value.table)
        var fieldslist = await table.getFieldMetaList()
        var name_list = []
        for(const field of fieldslist){
          name_list.push(field.name)
        }

        const recordList = await table.getRecordList();

        
        for(const fieldname of selected_options.value){
          var field = await table.getFieldByName(fieldname)
            
          var newname = fieldname+addname.value

          if(name_list.indexOf(newname)==-1){
            const r = await table.addField({type:1,name:newname})
          }
          const namefield = await table.getFieldByName(newname)
          for (const record of recordList) {
            const val = await field.getValue(record.id);
            var filename = []
            if(val){
              for(var file of val){
                filename.push(file.name)
              }
              filename = filename.join(', ')
              namefield.setValue(record.id,filename)
            }
          }
        }
        localStorage.setItem('selected_options', selected_options.value.join(','))
        await bitable.ui.showToast({
          toastType: "success",
          message: i18n.global.t('ygx')
        })
      };

      onMounted(async () => {
         
        const [tableList, selection] = await Promise.all([bitable.base.getTableMetaList(), bitable.base.getSelection()]);
        formData.value.table = selection.tableId;
        tableMetaList.value = tableList;


        const table = await bitable.base.getTableById(formData.value.table);
        const attachmentFieldList = await table.getFieldListByType(FieldType.Attachment);
        const recordList = await table.getRecordList();
        var options_name = []
        for(var i=0;i<attachmentFieldList.length;i++){
          const af = attachmentFieldList[i]
          const data = await table.getFieldMetaById(af.id)
          options.value.push({value:data.name,label:data.name})
          options_name.push(data.name)
        }
        if (localStorage.getItem('selected_options') !== null) {
          var storage_selected_options = localStorage.getItem('selected_options').split(',')
          var load_options = []
          for(var name of storage_selected_options){
            if(options_name.indexOf(name)!=-1){
              load_options.push(name)
            }
          }
          selected_options.value = load_options
        }

      });
      
      return {
        formData,
        tableMetaList,
        options,
        selected_options,
        addname,
        exportall,
        reloadOptions
      };
    },
  };
</script>

<template>
    <el-form ref="form" class="form" :model="formData" label-position="top" >
    <el-alert style="margin-bottom: 20px;background-color: #e1eaff;color: #606266;"  :title="$t('xzs')" type="info" />
    <el-alert style="margin-bottom: 20px;"  :title="$t('yyq')" type="warning" />
    <el-form-item :label="$t('qxz')" size="large">
        <el-select v-model="formData.table" :placeholder="$t('qxz')" style="width: 100%" @change='reloadOptions'>
          <el-option
            v-for="meta in tableMetaList"
            :key="meta.id"
            :label="meta.name"
            :value="meta.id"
          />
        </el-select>
    </el-form-item>
    <el-form-item :label="$t('xzx')" size="large">
      <el-select v-model="selected_options" multiple :placeholder="$t('xzx')" style="width: 100%">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
    </el-form-item>
    <el-form-item :label="$t('xzd')" size="large" >
      <el-input v-model="addname" placeholder=""></el-input>
    </el-form-item>
    <el-button type="primary" plain size="large" @click="exportall" style='width:100%'>{{$t('dc')}}</el-button>
  </el-form>
</template>

<style scoped>
  .form :deep(.el-form-item__label) {
    font-size: 16px;
    color: var(--el-text-color-primary);
    margin-bottom: 0;
  }
  .form :deep(.el-form-item__content) {
    font-size: 16px;
  }
</style>
