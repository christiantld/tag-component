<template>
  <TagInput
    v-model="tags"
    :options="['Hello', 'World']"
    :allowCustom="true"
    :showCount="true"
  />
</template>

<script lang="ts">
  import TagInput from "./components/TagInput.vue";

  export default {
    name: "App",
    components: {
      TagInput
    },
    data() {
      return {
        tags: []
      };
    }
  };
</script>
