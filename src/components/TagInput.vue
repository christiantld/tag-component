<template>
  <div class="tag-input">
    <input
      v-model="newTag"
      type="text"
      id="newTag"
      @keydown.enter="addTag(newTag)"
      @keydown.prevent.tab="addTag(newTag)"
      @keydown.delete="newTag.length || removeTag(tags.length - 1)"
      :style="{ 'padding-left': `${paddingLeft}px` }"
    />
    <ul class="tags" ref="tagsUl">
      <li v-for="(tag, index) in tags" :key="index" class="tag">
        {{ tag }}
        <button class="delete" @click="removeTag(index)">&times;</button>
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
  import {
    defineComponent,
    Ref,
    ref,
    nextTick,
    watch,
    onMounted,
    PropType
  } from "vue";

  interface SetupReturn {
    tags: Ref<string[]>;
    newTag: Ref<string>;
    paddingLeft: Ref<number>;
    tagsUl: Ref<number | null>;
    addTag(tag: string): void;
    removeTag(index: number): void;
    onTagsChange(): void;
  }

  export default defineComponent({
    props: {
      modelValue: { type: Array as PropType<string[]>, default: () => [] }
    },
    setup(props, { emit }): SetupReturn {
      const tags = ref(props.modelValue);
      const newTag = ref("");
      const paddingLeft = ref(10);
      const tagsUl = ref(null);

      function addTag(tag) {
        tags.value.push(tag);
        newTag.value = "";
      }
      function removeTag(index) {
        tags.value.splice(index, 1);
      }

      function onTagsChange() {
        const extraCushion = 15;
        paddingLeft.value = tagsUl.value.clientWidth + extraCushion;
        tagsUl.value.scrollTo(tagsUl.value.scrollWidth, 0);
        emit("update:modelValue", tags.value);
      }

      watch(tags, () => nextTick(onTagsChange), { deep: true });
      onMounted(onTagsChange);

      return {
        tags,
        newTag,
        paddingLeft,
        tagsUl,
        addTag,
        removeTag,
        onTagsChange
      };
    }
  });
</script>

<style scoped>
  input {
    width: 100%;
    padding: 10px 0;
  }

  .delete {
    color: white;
    background: none;
    outline: none;
    border: none;
    cursor: pointer;
    font-size: 16px;
  }

  .tag-input {
    position: relative;
  }

  ul {
    list-style: none;
    display: flex;
    align-items: center;
    gap: 7px;
    margin: 0;
    padding: 0;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 10px;
    max-width: 75%;
    overflow-x: auto;
  }
  .tag {
    background: rgb(250, 104, 104);
    padding: 5px;
    border-radius: 4px;
    color: white;
    white-space: nowrap;
    transition: 0.1s ease background;
  }
</style>
