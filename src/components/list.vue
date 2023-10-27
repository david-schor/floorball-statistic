<template>
  <div class="box">
    <h1 class="title is-4 mb-2">{{ title }}</h1>
    <div class="list has-visible-pointer-controls">
      <div class="list-item" v-for="(entry, index) in object" :key="index">
        <div class="list-item-image">
          <figure class="image is-64x64">
            <img
              class="is-rounded"
              src="https://via.placeholder.com/128x128.png?text=image"
            />
          </figure>
        </div>
        <div class="list-item-content">
          <div class="list-item-title">{{ Object.values(entry)[0] }}</div>
          <div class="list-item-description">
            <div
              class="tag is-rounded"
              v-for="(tag, tagIndex) in Object.values(entry).slice(1, -2)"
              :key="tagIndex"
            >
              {{ tag }}
            </div>
          </div>
        </div>
        <div class="list-item-controls">
          <div class="buttons is-right">
            <button
              class="button"
              v-for="(child, childIndex) in childs"
              :key="childIndex"
              @click="callFunction(child, entry)"
            >
              <span class="icon is-small">
                <i :class="child.icon"></i>
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const { title, object, childs } = defineProps({
  title: String,
  object: Array,
  childs: Array,
});

const callFunction = (child, entry) => {
  if (typeof child.func === "function") {
    child.func(entry);
  }
};
</script>
