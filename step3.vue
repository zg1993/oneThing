<template>
  <div class="signature">
    <vue-esign v-model:modelValue="img" />
    <van-button round block type="primary" style="margin-top: 12px" @click="onPrev">上一步</van-button>
    <van-button round block type="primary" style="margin-top: 12px" @click="onClick">提交</van-button>
  </div>
</template>

<script lang="ts" setup>
  import { ref, PropType } from 'vue';
  import VueEsign from '@/components/Esign/index.vue';
  import { showLoading, hideLoading, showWarning, showError } from '@/hooks/useMessage';
  import { uploadImage } from '@/api/app';
  import { employeeWordFile } from '@/api/oneThing';
  import { base64ImgtoFile } from '@/utils/util';
  import dayjs from 'dayjs';

  const props = defineProps({
    modelValue: {
      type: Object,
      default: () => {},
    },
    formState: {
      type: Object as PropType<any>,
      default: () => {},
    },
  });

  const emits = defineEmits(['nextStep', 'preStep', 'update:modelValue']);

  const img = ref('');

  // 上一步
  const onPrev = () => {
    emits('preStep');
  };

  const onClick = async () => {
    showLoading('提交中');
    let signUrl = await submitSign();
    if (signUrl) {
      await submitWord(signUrl);
    }
    hideLoading();
  };

  // 提交签名
  const submitSign = async () => {
    if (!img.value) {
      showWarning('签名为空！');
    }
    const formData: any = new FormData();
    formData.append('file', base64ImgtoFile(img.value));
    const [err, res] = await uploadImage(formData);
    if (err) {
      showError(err);
      return;
    }
    emits('update:modelValue', res.fileUrl);
    return res.fileUrl;
  };

  const submitWord = async (signUrl: string) => {
    let today = dayjs(new Date()).format('YYYY年MM月DD日');
    const formData = { applyTime: today, applySignature: signUrl, ...props.formState };
    const [err, res] = await employeeWordFile(formData);
    if (err) {
      showError('事项提交失败，请重试！');
    } else {
      // 提交formList附件信息需要
    }
  };

  // const submit;
</script>

<style scoped lang="less">
  .signature {
    margin-top: 10px;
    padding: 12px;
    background-color: #fff;
    .sign-class {
      width: 100%;
      border: 2px dashed #1989fa;
    }
  }
</style>
