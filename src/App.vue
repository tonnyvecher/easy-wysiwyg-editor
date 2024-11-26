<template>
  <div id="block-editor">
    <div class="toolbar">
      <button @click="undo"><img src="/src/images/undo.svg" alt="" /></button>
      <button @click="redo"><img src="/src/images/redo.svg" alt="" /></button>
      <button @click="addBlock('heading')">
        <img src="/src/images/h1.svg" alt="" />
      </button>
      <button @click="addBlock('paragraph')">
        <img src="/src/images/p.svg" alt="" />
        <img src="/src/images/p.svg" alt="" />
      </button>
      <button @click="addBlock('image')">
        <img src="/src/images/img.svg" alt="" />
      </button>
      <button @click="copyHTML">Скопировать HTML</button>
    </div>

    <div class="blocks">
      <div
        v-for="(block, index) in blocks"
        :key="block.id"
        class="block"
        :class="{ selected: selectedBlock === index }"
        ref="blockElements"
      >
        <div
          v-if="block.type === 'heading'"
          contenteditable="true"
          @input="updateBlock(index, $event)"
          class="heading"
          :data-placeholder="'Введите заголовок...'"
        >
          {{ block.content }}
        </div>
        <div
          v-if="block.type === 'paragraph'"
          contenteditable="true"
          @input="updateBlock(index, $event)"
          class="paragraph"
          :data-placeholder="'Введите текст...'"
        >
          {{ block.content }}
        </div>

        <div v-if="block.type === 'image'" class="image-block">
          <img :src="block.content" alt="Image Block" />
          <input
            type="text"
            placeholder="Введите URL изображения"
            @input="updateBlock(index, $event)"
            :value="block.content"
            class="url-input"
          />
        </div>
        <div class="block-controls">
          <button @click="moveBlock(index, 'up')">⬆️</button>
          <button @click="moveBlock(index, 'down')">⬇️</button>
          <button @click="removeBlock(index)">❌</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, watch, nextTick } from "vue";
import { nanoid } from "nanoid";

const historyIndex = ref(-1);
const history = ref([]);
const blocks = reactive([]);
const blockElements = ref([]);

const saveHistory = () => {
  if (historyIndex.value < history.value.length - 1) {
    history.value = history.value.slice(0, historyIndex.value + 1);
  }
  history.value.push(JSON.stringify(blocks));
  historyIndex.value++;
};

const restoreHistory = () => {
  const previousState = JSON.parse(history.value[historyIndex.value]);
  blocks.splice(0, blocks.length, ...previousState);
};

const undo = () => {
  if (historyIndex.value > 0) {
    historyIndex.value--;
    restoreHistory();
  }
};

const redo = () => {
  if (historyIndex.value < history.value.length - 1) {
    historyIndex.value++;
    restoreHistory();
  }
};

const addBlock = async (type) => {
  const newBlock = {
    id: nanoid(),
    type,
    content: "",
  };
  blocks.push(newBlock);
  saveHistory();

  await nextTick();
  const newBlockElement = blockElements.value[blocks.length - 1];
  if (newBlockElement) {
    const contentEditableElement = newBlockElement.querySelector(
      "[contenteditable='true']"
    );
    if (contentEditableElement) {
      contentEditableElement.focus();
    }
  }
};

const copyHTML = () => {
  let html = blocks
    .map((block) => {
      if (block.type === "heading") {
        return `<h1>${block.content}</h1>`;
      }
      if (block.type === "paragraph") {
        return `<p>${block.content}</p>`;
      }
      if (block.type === "image") {
        return `<img src="${block.content}" alt="Image Block"/>`;
      }
      return "";
    })
    .join("\n");

  navigator.clipboard.writeText(html).then(() => {
    alert("HTML скопирован в буфер обмена!");
  });
};

const updateBlock = (index, event) => {
  blocks[index].content = event.target.innerText || event.target.value;
  saveHistory();
};

const moveBlock = (index, direction) => {
  if (direction === "up" && index > 0) {
    const temp = blocks[index];
    blocks.splice(index, 1);
    blocks.splice(index - 1, 0, temp);
    saveHistory();
  } else if (direction === "down" && index < blocks.length - 1) {
    const temp = blocks[index];
    blocks.splice(index, 1);
    blocks.splice(index + 1, 0, temp);
    saveHistory();
  }
};

const removeBlock = (index) => {
  blocks.splice(index, 1);
  saveHistory();
};

watch(
  blocks,
  () => {
    if (history.value.length === 0) saveHistory();
  },
  { immediate: true }
);
</script>

<style>
#block-editor {
  max-width: 800px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

.toolbar {
  display: flex;
  gap: 10px;
  padding-bottom: 20px;
}

.toolbar button {
  padding: 5px 10px;
  background: #282828;
  cursor: pointer;
  border-radius: 4px;
  color: #007bff;
  transition: background 0.15s;
}

.toolbar button:hover {
  background: #eaeaea;
}

.blocks {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.block {
  padding: 10px;
  border-radius: 4px;
  position: relative;
}

.block.selected {
  border-color: #007bff;
}

.heading {
  font-size: 1.5em;
  font-weight: bold;
}

.paragraph {
  font-size: 1em;
}

.image-block img {
  max-width: 100%;
  height: auto;
  padding-bottom: 10px;
}

.block-controls {
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  gap: 5px;
}

.block-controls button {
  background: #f4f4f4;
  border: 1px solid #ccc;
  border-radius: 3px;
  padding: 3px 5px;
  cursor: pointer;
}

.block-controls button:hover {
  background: #eaeaea;
}

[data-placeholder]:empty::before {
  content: attr(data-placeholder);
  color: #d5c3c3;
  pointer-events: none;
}

.url-input {
  width: 100%;
  padding: 8px;
  font-size: 14px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.url-input:focus {
  border-color: #007bff;
  outline: none;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}
</style>
