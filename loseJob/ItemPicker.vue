<template>
  <div class="main">
    <item-checkbox :list="list" :title="title" @handle="handle" />
    <dialog-confirm ref="dialogRef" @confirm="goToIndex" />
  </div>
</template>

<script lang="ts" setup>
  import ItemCheckbox, { ICheckboxList } from '../../components/ItemCheckbox/index.vue';
  import icon0 from '@/assets/image/oneThing/blue.png';
  import { useRouter, useRoute } from 'vue-router';
  import { ref } from 'vue';
  import DialogConfirm from '@/components/DialogConfirm/index.vue';
  import { ItemEnum, title } from './modules/configEnums';

  const dialogRef = ref();

  const selectItem = ref<any[]>([]);

  const route = useRoute();

  const router = useRouter();

  const itemList = Object.values(ItemEnum);

  const list = ref<ICheckboxList[]>(
    Array.from({ length: itemList.length }, (_, i) => ({
      title: itemList[i],
      value: itemList[i],
      tip: '',
      icon: icon0,
    })),
  );

  //点击办理
  function handle(value: string[]) {
    selectItem.value = value;
    dialogRef.value.show();
  }

  const goToIndex = () => {
    router.push({ path: route.path + '/online', query: { itemName: JSON.stringify(selectItem.value) } });
  };
</script>

<style scoped lang="less">
  .main {
    height: 100%;
    overflow: auto;
  }
</style>
