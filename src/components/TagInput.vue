<template>
  <div class="tag-input">
    <input
      v-model="newTag"
      type="text"
      :list="id"
      autocomplete="off"
      @keydown.enter="addTag(newTag)"
      @keydown.prevent.tab="addTag(newTag)"
      @keydown.delete="newTag.length || removeTag(tags.length - 1)"
      :style="{ 'padding-left': `${paddingLeft}px` }"
    />

    <datalist v-if="options" :id="id">
      <option v-for="option in availableOptions" :key="option" :value="option">
        {{ option }}
      </option>
    </datalist>

    <ul class="tags" ref="tagsUl">
      <li
        v-for="(tag, index) in tags"
        :key="index"
        class="tag"
        :class="{ duplicate: tag === duplicate }"
      >
        {{ tag }}
        <button class="delete" @click="removeTag(index)">&times;</button>
      </li>
    </ul>
    <div v-if="showCount" class="count">
      <span>{{ tags.length }}</span> tags
    </div>
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
    PropType,
    computed,
    ComputedRef
  } from "vue";

  interface SetupReturn {
    tags: Ref<string[]>;
    newTag: Ref<string>;
    paddingLeft: Ref<number>;
    tagsUl: Ref<number | null>;
    id: string;
    duplicate: Ref<string | null>;
    addTag(tag: string): void;
    removeTag(index: number): void;
    onTagsChange(): void;
    availableOptions: ComputedRef<string[]>;
  }

  export default defineComponent({
    props: {
      modelValue: { type: Array as PropType<string[]>, default: () => [] },
      name: { type: String as PropType<string>, default: "" },
      options: {
        type: [Array, Boolean] as PropType<string[]>,
        default: false
      },
      allowCustom: { type: Boolean as PropType<boolean>, default: true },
      showCount: { type: Boolean as PropType<boolean>, default: false }
    },
    setup(props, { emit }): SetupReturn {
      const tags = ref(props.modelValue);
      const newTag = ref("");
      const paddingLeft = ref(10);
      const tagsUl = ref(null);
      const id = Math.random().toString(36).substring(7);
      const duplicate = ref(null);

      function addTag(tag: string) {
        if (!tag) return;
        if (!props.allowCustom && !props.options.includes(tag)) return;

        if (tags.value.includes(tag)) {
          handleDuplicate(tag);
          return;
        }

        tags.value.push(tag);
        newTag.value = "";
      }
      function removeTag(index: number) {
        tags.value.splice(index, 1);
      }

      function onTagsChange() {
        const extraCushion = 15;
        paddingLeft.value = tagsUl.value.clientWidth + extraCushion;
        tagsUl.value.scrollTo(tagsUl.value.scrollWidth, 0);
        emit("update:modelValue", tags.value);
      }

      function handleDuplicate(tag: string): void {
        duplicate.value = tag;
        setTimeout(() => (duplicate.value = null), 1000);
        newTag.value = "";
      }

      watch(tags, () => nextTick(onTagsChange), { deep: true });
      onMounted(onTagsChange);

      const availableOptions = computed(() => {
        if (!props.options) return [];
        return props.options.filter(option => !tags.value.includes(option));
      });

      return {
        tags,
        newTag,
        paddingLeft,
        tagsUl,
        id,
        duplicate,
        addTag,
        removeTag,
        onTagsChange,
        availableOptions
      };
    }
  });
</script>

<style scoped>
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
  input {
    width: 100%;
    padding: 10px;
  }
  .delete {
    color: white;
    background: none;
    outline: none;
    border: none;
    cursor: pointer;
  }
  @keyframes shake {
    10%,
    90% {
      transform: scale(0.9) translate3d(-1px, 0, 0);
    }

    20%,
    80% {
      transform: scale(0.9) translate3d(2px, 0, 0);
    }

    30%,
    50%,
    70% {
      transform: scale(0.9) translate3d(-4px, 0, 0);
    }

    40%,
    60% {
      transform: scale(0.9) translate3d(4px, 0, 0);
    }
  }
  .tag.duplicate {
    background: rgb(235, 27, 27);
    animation: shake 1s;
  }
  .count {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    right: 10px;
    display: block;
    font-size: 0.8rem;
    white-space: nowrap;
  }
  .count span {
    background: #eee;
    padding: 2px;
    border-radius: 2px;
  }
  .with-count input {
    padding-right: 60px;
  }
  .with-count ul {
    max-width: 60%;
  }
</style>
