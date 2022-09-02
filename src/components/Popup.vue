<script>
export default {
  props: ["title", "show"]
};
</script>

<template>
  <div v-if="show" class="backdrop">
    <div class="inner">
      <div>
        <h2>{{ title }}</h2>
      </div>
      <slot></slot>

      <div class="action">
        <slot name="action"></slot>
      </div>
    </div>
  </div>
</template>

<style scoped>
.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 15px;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 100;
}

.inner {
  background-color: white;
  max-width: 600px;
  border-radius: 5px;
  padding: 15px;
  margin: 5% auto 0 auto;
  display: grid;
  grid-template-rows: repeat(3, 1fr);
}

.action {
   margin-top: 15px;
  display: grid;
  grid-template-columns: 120px 100px;
  gap: 15px;
}
</style>
