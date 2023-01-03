<template>
  <div class="retire">
    <van-steps :active="active" active-icon="success" active-color="#38f" class="step">
      <van-step>填写信息</van-step>
      <van-step>材料上传</van-step>
      <van-step>签名</van-step>
      <van-step>提交成功</van-step>
    </van-steps>

    <div class="content">
      <step1
        v-show="active === 0"
        @preStep="preStep"
        @nextStep="nextStep"
        v-model="formState"
        @update:modelValue="updateFormState"
        v-model:itemCodeArray="itemCodeArray"
        :itemNameArray="itemNameArray"
      ></step1>
      <step2
        v-show="active === 1"
        @preStep="preStep"
        @nextStep="nextStep"
        :metail-list="metailList"
        :form-list="formList"
        :shiFouShuYuZhuanZhiQiYe="formState.shiFouShuYuZhuanZhiQiYe"
      ></step2>
      <step3 v-show="active === 2" @preStep="preStep" @nextStep="nextStep" v-model="imgBase64"></step3>
      <step4 v-show="active === 3" @preStep="preStep" @finish="finish" :submitResArray="submitResArray"></step4>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { ref, reactive } from 'vue';
  import { useRoute } from 'vue-router';
  import step1 from './modules/step1.vue';
  import step2 from './modules/step2.vue';
  import step3 from '@/components/OneThingPage/SignPage.vue';
  import step4 from '@/components/OneThingPage/Result.vue';
  // import { ItemFileKeyMap } from './modules/configEnums';
  // import type { TSubmit } from '@/api/gft-yc';
  import { getBaseInfo, IBaseInfo, getUploadMaterial, submit } from '@/api/gft-yc';
  import { usePromise } from '@/hooks/usePromise';
  import { noticeError } from '@/hooks/useNotice';
  import { showLoading, hideLoading, showWarning, showError } from '@/hooks/useMessage';
  // import { uploadImage } from '@/api/app';
  // import { employeeWordFile } from '@/api/oneThing';
  // import { base64ImgtoFile } from '@/utils/util';

  const route = useRoute();

  const { query } = route;

  const itemNameStr = query.itemName as string;

  const itemNameArray = ref<string[]>(JSON.parse(itemNameStr));

  const itemCodeArray = ref<string[]>([]);

  const imgBase64 = ref<any>('');

  const active = ref(0);

  const formState = reactive<any>({});

  const submitResArray = ref<any[]>([]);

  const formList = ref<any[]>();
  const metailList = ref<any[]>();

  const nextStep = async () => {
    // 自动生成附件
    if (0 === active.value) {
      metailList.value = [];
      await getFormList();
    }

    if (2 === active.value) {
      showLoading('提交中');
      await submitYC(formList.value); // 提交表单操作
      hideLoading();
    }
    if (active.value < 3) {
      active.value += 1;
    }
  };

  const preStep = () => {
    if (active.value > 0) {
      active.value -= 1;
    }
  };

  const finish = () => {};

  const getFormList = async () => {
    try {
      showLoading('加载中');
      [formList.value, metailList.value] = await createFormList();
    } catch (e) {
      console.log(e);
      let msg = String(e).slice(7);
      if (msg) {
        noticeError(msg);
      }
      return false;
    } finally {
      console.log('----');
      hideLoading();
    }
    return true;
  };

  // const generateYeWuBeiZhuJSON = () => {
  //   let obj = {
  //     gongZuoDanWei: '',
  //     gangWeiLeiBie: '',
  //     shiFouShuYuZhuanZhiQiYe: '',
  //     shiFouYiJiaoGeRenDangAn: '',
  //   };
  //   Object.keys(obj).forEach((key: string) => {
  //     if (formState[key]) {
  //       obj[key] = formState[key];
  //     }
  //   });
  //   return JSON.stringify(obj);
  // };

  const createFormList = async () => {
    let formList: any[] = [];
    let metailList: any[] = [];
    const requests: any[] = [];

    const requestsDoc: any[] = [];
    console.log('itemCodeArray');
    console.log(itemCodeArray.value);
    itemCodeArray.value.forEach((code: string) => {
      requests.push(getBaseInfo(code));
      requestsDoc.push(getUploadMaterial(code));
    });
    let err, resInfoList: any, errDoc, resDocList: any;
    [err, resInfoList] = await usePromise(Promise.all(requests));
    [errDoc, resDocList] = await usePromise(Promise.all(requestsDoc));
    if (err || errDoc) {
      throw new Error('基本信息获取失败，请重试！');
    }
    // console.log(err);
    // console.log(errDoc);
    resInfoList.forEach((base: any, index: number) => {
      // let itemName = itemNameArray.value[index];
      // let filePath = generateFilePath(itemName, docUrl);

      let metail = resDocList[index];
      formList.push({
        ...base,
        metailList: metail,
        // formData: { linkAddress: formState.linkAddress, YeWuBeiZhu: generateYeWuBeiZhuJSON() },
        formData: { linkAddress: formState.linkAddress },
      });

      metail.forEach((file: any) => {
        file.FILE_NAME = file.DOCUMENT_NAME;
        file.FILE_PATH = '';
        metailList.push(file);
        // file.FILE_PATH = filePath;
      });
    });
    return [formList, metailList];
  };

  const submitYC = async (formList) => {
    const requests: any[] = [];
    console.log('formlist: ');
    console.log(formList);
    formList.forEach((formData: any) => {
      requests.push(submit(formData));
    });
    console.log('requests: ');
    console.log(requests);
    const resList = await Promise.all(requests);
    // let resList = [];
    submitResArray.value = handleResYC(resList);
  };

  const handleResYC = (resList: any[]) => {
    let result: string[] = [];
    resList.forEach((i: any, index: number) => {
      // 返回结果对应关联事项的名称
      let itemName = itemNameArray.value[index];
      let resTip = '';
      if ('200' == i.state) {
        resTip = '提交成功！';
      } else if (i.error && '300' == i.state && i.error.includes('重复')) {
        resTip = i.error;
      } else {
        resTip = '事项提交失败，请重试!';
      }
      result.push(itemName + '：' + resTip);
    });
    return result;
  };

  const updateFormState = (value: any) => {
    Object.keys(value).forEach((key: string) => {
      formState[key] = value[key];
    });
  };
</script>

<style lang="less" scoped>
  .retire {
    overflow: auto;
    height: 100%;
  }
  // .content {
  //   :deep(.step) {
  //     .button {
  //       // margin: 16px;
  //     }
  //   }
  // }

  .van-steps {
    padding: 20px 20px 10px;
  }
</style>
