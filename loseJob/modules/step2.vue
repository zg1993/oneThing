<template>
  <div class="gjj-fileinfo">
    <van-form label-width="100%" @submit="onSubmit" @failed="onFailed">
      <van-cell-group title="上传材料">
        <van-field
          v-for="(item, key) in metailList"
          :key="key"
          :label="displayLabel(item.DOCUMENT_NAME)"
          :required="item.MUST"
          :name="dropList.includes(item.DOCUMENT_NAME) ? '' : 'FILE_PATH'"
          readonly
          :rules="[
            {
              required: item.MUST && !dropList.includes(item.DOCUMENT_NAME),
              message: `请上传${displayLabel(item.DOCUMENT_NAME)}`,
            },
          ]"
          v-show="!dropList.includes(item.DOCUMENT_NAME)"
        >
          <template #input>
            <upload-img-l-c v-model="item.FILE_PATH" :maxCount="3"></upload-img-l-c>
          </template>
        </van-field>
      </van-cell-group>

      <div class="button">
        <van-button round block @click="onPrev" style="margin-bottom: 12px">上一步</van-button>
        <van-button round block type="primary" native-type="submit">下一步</van-button>
      </div>
    </van-form>
    <check-user-card ref="checkCardRef"></check-user-card>
  </div>
</template>

<script lang="ts" setup>
  import { ref, PropType, reactive, onMounted, computed, watch } from 'vue';
  import UploadImgLC from '@/components/UploadFileLC/index.vue';
  import CheckUserCard from '@/components/CheckUserCard/index.vue';
  import { RemoveDuplication } from '@/utils/utilsClass';

  const formStateFile = reactive({
    wordDocument: '',
    idCard: '',
    huKouBen: '',
    sheHuiBaoZhangKa: '',
    qiYeGaiZhiXieYiShu: '',
  });

  const emits = defineEmits(['nextStep', 'preStep', 'update:modelValue']);

  const itemCodeList = ref<string[]>([]);

  const props = defineProps({
    metailList: {
      type: Array as PropType<any[]>,
      default: () => [],
    },

    formList: {
      type: Array as PropType<any[]>,
      default: () => [],
    },
  });
  // 去重相关
  const duplicationMap = {
    身份证正反面照片: [
      '身份证',
      '申请人的身份证或户口簿',
      '申请人的身份证或社会保障卡',
      '身份证或社保卡',
      '身份证或社会保障卡',
    ],
  };

  const rd = new RemoveDuplication([], duplicationMap);

  const dropList = ref<string[]>([]);

  const displayLabel = (labelName: string) => {
    return rd.displayLabelMap[labelName] || labelName;
  };

  // 去重处理
  watch(
    () => props.metailList,
    () => {
      // console.log(props.metailList);
      rd.settingMetailList(props.metailList);
      dropList.value = rd.handle();
    },
    { immediate: true },
  );

  const checkCardRef = ref();

  const onPrev = () => {
    emits('preStep');
  };

  const onSubmit = async () => {
    console.log(props.metailList);
    console.log(props.formList);
    // 恢复
    rd.reset(dropList.value);
    console.log(props.formList);
    // emits('update:modelValue', formStateFile);
    emits('nextStep');
  };

  const onFailed = (error) => {
    console.log(error);
    console.log(props.metailList);
    console.log(props.formList);
    // console.log(formStateFile);
    // 恢复
    rd.reset(dropList.value);
    console.log(props.formList);
  };
</script>

<style lang="less" scoped>
  .gjj-fileinfo .van-cell {
    flex-wrap: wrap;
  }
  .gjj-fileinfo .van-field__label {
    width: 100%;
  }
  .hint {
    text-align: center;
    color: @text-color-grey;
    padding: 60px 0;
  }

  .button {
    box-sizing: border-box;
    position: fixed;
    bottom: 40px;
    width: 100%;
    padding: 10px @spacing-col-base 0;
    background-color: #fff;
  }

  .gjj-fileinfo {
    margin-bottom: calc(var(--van-button-default-height) * 2 + 20px + 12px);
  }

  .download-link {
    color: #2d8cfc;
  }
</style>
