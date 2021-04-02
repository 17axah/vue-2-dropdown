<template>
  <div>
    <div>APP</div>

    <Dropdown>
      <div>trigger</div>

      <template #portal>
        <div>PORTAL</div>
      </template>
    </Dropdown>
  </div>
</template>

<script>
import Dropdown from '@/components/Dropdown.vue';

export default {
  components: {
    Dropdown,
  },
};
</script>
