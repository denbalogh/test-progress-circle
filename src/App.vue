<template>
  <div id="app">
    <div class="panel">
      <ProgressCircle :current-value="0" :max-value="100" />
      <ProgressCircle :current-value="39" :max-value="100" />
      <ProgressCircle :current-value="13" :max-value="37" />
      <ProgressCircle :current-value="8" :max-value="15" :label="'Label'" />
    </div>
    <div class="panel">
      <ProgressCircle :size="'compact'" :current-value="0" :max-value="100" />
      <ProgressCircle :size="'compact'" :current-value="42" :max-value="100" />
      <ProgressCircle
        :label="'Label'"
        :size="'compact'"
        :current-value="42"
        :max-value="73"
      />
    </div>
  </div>
</template>

<script>
import ProgressCircle from "./components/ProgressCircle.vue";

export default {
  name: "App",
  components: {
    ProgressCircle
  }
};
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
}
.panel {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}
</style>
