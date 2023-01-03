<template>
  <div class="step">
    <van-form @submit="onSubmit" label-width="120px" colon @failed="onFailed">
      <!-- 14 -->
      <van-cell-group title="基本信息">
        <van-field
          label="姓名"
          v-model="applyNameDesensitized"
          required
          readonly
          placeholder="请输入"
          maxlength="20"
        ></van-field>
        <van-field
          label="身份证号"
          v-model="idCardNumDesensitized"
          placeholder="请输入"
          required
          readonly
          maxLength="18"
        ></van-field>
        <van-field
          label="联系方式"
          v-model="applyPhoneDesensitized"
          placeholder="请输入"
          required
          readonly
          maxlength="11"
        ></van-field>
        <van-field
          label="地址"
          v-model="formState.linkAddress"
          placeholder="请输入"
          required
          maxlength="50"
          :rules="[{ required: true, message: '请输入地址' }]"
        ></van-field>
      </van-cell-group>

      <van-cell-group :title="ItemEnum.JYGNRYRD" v-if="itemNameArray.includes(ItemEnum.JYGNRYRD)">
        <popup-picker
          label-width="120px"
          :columns="employmentDifficultiesColumns"
          name="employmentDifficultiesAddress"
          label="办理地"
          v-model="formState.employmentDifficultiesAddress"
          placeholder="请选择"
          maxlength="50"
          @change="onChangeAreaColumns($event, ItemEnum.JYGNRYRD)"
          required
          :rules="[{ required: true, message: '请选择办理地' }]"
        ></popup-picker>
      </van-cell-group>
      <van-cell-group title="失业保险金申领" v-if="itemNameArray.includes(ItemEnum.SYBXJSL)">
        <popup-picker
          label-width="120px"
          :columns="loseJobColumns"
          name="loseJobAddress"
          label="办理地"
          v-model="formState.loseJobAddress"
          placeholder="请选择"
          maxlength="50"
          @change="onChangeAreaColumns($event, ItemEnum.SYBXJSL)"
          required
          :rules="[{ required: true, message: '请选择办理地' }]"
        ></popup-picker>
      </van-cell-group>

      <div class="button">
        <van-button type="primary" native-type="submit" block round>下一步</van-button>
      </div>
    </van-form>
  </div>
</template>

<script lang="ts" setup>
  import { reactive, PropType, onMounted, ref, computed } from 'vue';
  import { Desensitized } from '@/utils/util';
  import PopupPicker from '@/components/PopupPicker/index.vue';
  import { useUserStore } from '@/store/modules/user';
  import { ItemEnum, itemCodeRegionMap } from './configEnums';
  // import { commonWhetherColumns, whetherMedicalInsuranceColumns } from '@/utils/pickerColumns';

  const userStore = useUserStore();

  // const intersectColumns = ref<any[]>([]); // 交集数组 12-12不取交集
  const employmentDifficultiesColumns = ref<any[]>([]);
  const loseJobColumns = ref<any[]>([]);

  const emits = defineEmits(['nextStep', 'update:modelValue', 'update:itemCodeArray']);

  const applyNameDesensitized = Desensitized.desensitizedName(userStore.userInfo.name);
  const idCardNumDesensitized = Desensitized.desensitizedIdNo(userStore.userInfo.cardid);
  const applyPhoneDesensitized = Desensitized.desensitizedPhone(userStore.userInfo.mobile);

  const formState = reactive({
    //basic
    applyName: userStore.userInfo.name,
    idCardNum: userStore.userInfo.cardid,
    applyPhone: userStore.userInfo.mobile,
    linkAddress: '',

    employmentDifficultiesAddress: '',
    loseJobAddress: '',
  });

  const props = defineProps({
    modelValue: {
      type: Object as PropType<any>,
      required: true,
    },
    itemCodeArray: {
      type: Array,
      default: () => [],
    },
    itemNameArray: {
      type: Array as PropType<string[]>,
      default: () => [],
    },
  });

  onMounted(() => {
    initAreaColumns();
  });

  const onSubmit = async () => {
    emits('update:modelValue', formState);
    emits('nextStep');
  };

  const initAreaColumns = () => {
    employmentDifficultiesColumns.value = intersectAreaColumns(props.itemNameArray, [ItemEnum.JYGNRYRD]);
    loseJobColumns.value = intersectAreaColumns(props.itemNameArray, [ItemEnum.SYBXJSL]);
  };

  // 地区取交集，itemNameArray选择的事项， rangeItemName需要取交集的事项
  const intersectAreaColumns = (itemNameArray: string[], rangeItemName: string[]) => {
    let intersectItemName = itemNameArray.filter((val: string) => rangeItemName.includes(val));
    let intersect: any[] = [];
    intersectItemName.forEach((itemName: string, index: number) => {
      let regions = itemCodeRegionMap[itemName];
      if (regions) {
        let columns: any[] = Object.keys(regions);
        if (0 === index) {
          intersect = columns;
        } else {
          intersect = intersect.filter((val: string) => columns.includes(val));
        }
      }
    });
    return intersect;
  };

  const itemCodeArray = new Array(props.itemNameArray.length).fill('');

  const onChangeAreaColumns = (areaName: any, itemName: any) => {
    //就业困难办理地 employmentDifficultiesAddress
    //失业保险金申领办理地 loseJobAddress
    let index = props.itemNameArray.indexOf(itemName);
    if (-1 === index) {
      console.log('error!');
      return;
    }
    let regions = itemCodeRegionMap[itemName];
    if (regions) {
      let itemCode = regions[areaName];
      itemCodeArray[index] = itemCode;
    }
    emits('update:itemCodeArray', itemCodeArray);
  };

  const onFailed = (errorInfo) => {
    console.log('-----failed');
    console.log(errorInfo);
    console.log(formState);
  };
</script>

<style lang="less" scoped>
  .step {
    margin-bottom: calc(var(--van-button-default-height) + 20px);
  }
  .button {
    box-sizing: border-box;
    position: fixed;
    bottom: 40px;
    width: 100%;
    padding: 10px @spacing-col-base 0;
    background-color: #fff;
  }
</style>
