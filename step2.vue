<template>
  <div class="gjj-fileinfo">
    <van-form colon label-width="100%" @submit="onSubmit" @failed="onFailed">
      <van-cell-group title="上传材料">
        <van-field
          label="身份证正反面照片"
          required
          name="FILE_PATH"
          readonly
          :rules="[{ required: true, message: '请上传身份证正反面照片' }]"
        >
          <template #input>
            <upload-img-l-c v-model="url" :maxCount="2"></upload-img-l-c>
          </template>
        </van-field>
      </van-cell-group>

      <div style="margin: 16px">
        <van-button round block @click="onPrev" style="margin-bottom: 12px">上一步</van-button>
        <van-button round block type="primary" native-type="submit">下一步</van-button>
      </div>
    </van-form>
    <check-user-card ref="checkCardRef"></check-user-card>
  </div>
</template>

<script lang="ts" setup>
  import { onMounted, reactive, ref, onActivated, PropType } from 'vue';
  import UploadImgLC from '@/components/UploadFileLC/index.vue';
  import CheckUserCard from '@/components/CheckUserCard/index.vue';
  import { showLoading, hideLoading, showWarning } from '@/hooks/useMessage';
  import { getBaseInfo, IBaseInfo, getUploadMaterial, submit } from '@/api/gft-yc';
  import { usePromise } from '@/hooks/usePromise';
  import { noticeError } from '@/hooks/useNotice';

  const url = ref('');

  let baseInfo: any = {};

  const emits = defineEmits(['nextStep', 'prevStep', 'update:modelValue']);
  const props = defineProps({
    modelValue: {
      type: Array as PropType<any[]>,
      default: () => [],
    },
    formState: {
      type: Object as PropType<any>,
      required: true,
    },
    // data: {
    //   type: Object,
    //   required: true,
    // },
    itemCodeArray: {
      type: Array as PropType<string[]>,
      default: () => [],
    },
  });

  const fileList = ref<any[]>([]);

  onActivated(() => {});
  const checkCardRef = ref();

  const onPrev = () => {
    emits('prevStep');
  };
  const onSubmit = async () => {
    showLoading('加载中');
    const res = await generateFormList();
    console.log(res);
    console.log('-----------------------------------------');
    hideLoading();
    if (res) {
      emits('nextStep');
    }
    console.log(props.itemCodeArray);
  };

  const onFailed = () => {
    // console.log(props.itemCodeArray);
    console.log(props.formState);
  };

  const generateFormList = async () => {
    let formList: any[] = [];
    const requests: any[] = [];
    const requestsDoc: any[] = [];
    props.itemCodeArray.forEach((code: string) => {
      requests.push(getBaseInfo(code));
      requestsDoc.push(getUploadMaterial(code));
    });
    // const resInfoList = await Promise.all(requests);
    // const resDocList = await Promise.all(requestsDoc);
    let err, resInfoList: any, errDoc, resDocList: any;

    [err, resInfoList] = await usePromise(Promise.all(requests));
    [errDoc, resDocList] = await usePromise(Promise.all(requestsDoc));

    if (err || errDoc) {
      noticeError('基本信息获取失败，请重试！');
      return;
    }
    // console.log(err);
    // console.log(errDoc);
    resInfoList.forEach((base: any, index: number) => {
      let metail = resDocList[index];
      // formList.value.push({
      formList.push({
        ...base,
        metailList: metail,
        formData: { linkAddress: props.formState.liveAddress },
      });

      metail.forEach((file: any) => {
        file.FILE_NAME = file.DOCUMENT_NAME;
        file.FILE_PATH = '';
        // metailList.value.push(file);
      });
    });
    emits('update:modelValue', formList);

    // formList.forEach((i) => {
    //   let name = i.ItemName;
    //   console.log(name);
    //   i.metailList.forEach((d) => {
    //     let fileName = d.DOCUMENT_NAME;
    //     let text = d.MUST ? '必传' : '不必传';
    //     console.log(`${fileName}: ${text}`);
    //   });
    //   console.log('---------------');
    // });

    return true;
    // console.log(metailList.value);
    // console.log('formList');
  };
</script>

<style lang="less">
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
</style>
