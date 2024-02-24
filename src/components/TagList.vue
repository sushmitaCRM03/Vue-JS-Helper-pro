<template>
  <ul class="tag-list">
    <li
      class="tag-default tag-pill tag-outline"
      v-for="(tag, index) of tags"
      :key="index"
    >
      <span v-text="tag" />
    </li>
  </ul>
  <div></div>
</template>


<script>
import { mapGetters } from "vuex";
export default {
  name: "TagList",
  props: {
    tags: Array
  }
};
</script>
