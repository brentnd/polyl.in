<template>
  <div class="input-group">
    <input
      class="form-control"
      type="text"
      v-model="displayValue"
      @blur="isInputActive = false"
      @focus="isInputActive = true"
    >
    <div class="input-group-append">
      <div class="input-group-text">{{ units }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: "DistanceInput",
  props: ["value", "units"],
  data: function() {
    return {
      isInputActive: false,
      newValue: 0
    };
  },
  computed: {
    displayValue: {
      get: function() {
        if (this.isInputActive && this.value == 0.0) {
          return "";
        }
        return this.value.toFixed(2);
      },
      set: function(modifiedValue) {
        this.newValue = parseFloat(modifiedValue.replace(/[^\d\.]/g, ""));
        if (isNaN(this.newValue)) {
          this.newValue = 0;
        }
      }
    }
  },
  watch: {
    isInputActive: function(val) {
      if (!val) {
        this.$emit("input", this.newValue);
      }
    }
  }
};
</script>

<style scoped>
</style>
