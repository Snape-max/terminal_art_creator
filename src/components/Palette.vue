<template>
    <a-color-picker 
    v-model:modelValue="selectedColor"
    defaultValue="#000000" 
    hideTrigger 
    showPreset
    @change="handleColorChange"
    />
</template>


<script>
export default {
  name: 'Palette',
  data() {
    return {
      selectedColor: '#000000'
    };
  },
  methods: {
    handleColorChange(color) {
      this.$emit('color-selected', color);
    }
  }
}
</script>


<style scoped>
.color-picker {
  padding: 16px;
}
</style>