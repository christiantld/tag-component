<template>
  <TagInput v-model="tags" />
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
