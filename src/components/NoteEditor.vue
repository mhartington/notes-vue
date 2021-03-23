<template>
  <div class="quill-editor">
    <slot name="toolbar"></slot>
    <div ref="editor" class="user-editor"></div>
  </div>
</template>
<script lang="ts">
import 'quill/dist/quill.core.css';
import 'quill/dist/quill.snow.css';

import { defineComponent, onMounted, ref, watch } from 'vue';
import Quill from 'quill';
export default defineComponent({
  name: 'NoteEditor',
  props: {
    content: String,
    modelValue: String,
  },
  setup(props, { emit }) {
    const editor = ref<HTMLElement | null>(null);
    const _content = ref(props.content);
    let quill: Quill;

    const initQuill = () => {
      if (editor.value) {
        quill = new Quill(editor.value, {
          theme: 'snow',
          modules: {
            toolbar: [
              [{ header: [1, 2, 3, 4, 5, 6, false] }],
              ['bold', 'italic', 'underline', 'strike'],
              [{ list: 'ordered' }, { list: 'bullet' }],
              [{ indent: '-1' }, { indent: '+1' }],
            ],
          },
        });

        quill.on('text-change', () => {
          const html = quill.root.innerHTML;
          _content.value = html;
          emit('update:modelValue', html);
          emit('input', _content.value);
        });
      }
    };
    const setHtml = (content: string) => {
      quill.clipboard.dangerouslyPasteHTML(content);
      quill.blur();
    };
    watch(
      () => props.content,
      (newValue) => {
        if (newValue && newValue !== _content.value) {
          _content.value = newValue;
          setHtml(newValue);
        }
      }
    );
    watch(
      () => props.modelValue,
      (newValue) => {
        if (newValue && newValue !== _content.value) {
          _content.value = newValue;
          setHtml(newValue);
        }
      }
    );
    onMounted(() => initQuill());
    return { editor };
  },
});
</script>
<style>
.quill-editor {
  display: flex;
  flex-direction: column;
  height: 100%;
}
.ql-toolbar {
  border: none;
}
.ql-toolbar.ql-snow,
.ql-container.ql-snow {
  border: none;
}
.user-editor {
  flex: 1 1 auto;
  border: none;
  font-size: unset;
}
</style>
