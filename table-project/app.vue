<template>
  <div>
    <NuxtLayout name="default">
      <Tables />
    </NuxtLayout>
  </div>
</template>

<script>
import Tables from '@/components/tables.vue';

export default defineComponent({
  layout: 'default',
  store: true,
  components:{
    Tables
  }
})
</script>

