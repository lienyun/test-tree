<template lang="">
  <v-list-group :value="model.groupId" v-if="isFolder">
    <template v-slot:activator="{ props }">
      <v-list-item
        v-bind="props"
        :title="model.groupName"
        class="pa-0"
        :prepend-icon="model.root? 'mdi mdi-domain':''"
        v-if="!model.parentGroupId"
      ></v-list-item>

      <v-list-item v-else v-bind="props" class="pa-0">
        <template v-slot:prepend="{ isActive }">
          <v-list-item-action start>
            <v-checkbox-btn
              v-model="selectAll"
              :true-value="{ model, isSelected: true }"
              :false-value="{ model, isSelected: false }"
              @update:modelValue="selectAllFunc"
            ></v-checkbox-btn
            >{{ selectAll.isSelected }}
          </v-list-item-action>
        </template>

        <v-list-item-title>{{ model.groupName }}</v-list-item-title>
      </v-list-item>
    </template>

    <TestGroupTree
      v-for="children in model.groupTreeNodes"
      :key="children.groupId"
      :model="children"
      :defaultCheckbox="defaultCheckbox"
      @onCheckbox="handleCheckbox"
    />
  </v-list-group>

  <v-list-item v-else>
    <template v-slot:prepend="{ isActive }">
      <v-list-item-action start>
        <v-checkbox-btn
          v-model="checkbox"
          :true-value="{ model, isSelected: true }"
          :false-value="{ model, isSelected: false }"
          @update:modelValue="emitCheckbox"
        ></v-checkbox-btn>
      </v-list-item-action>
    </template>

    <v-list-item-title>{{ model.groupName }}</v-list-item-title>
  </v-list-item>
</template>
<script setup>
import { computed, ref, onMounted, watch, watchEffect } from "vue";
const props = defineProps({
  model: Object,
  defaultCheckbox: Array,
  allItems: Array,
  selectAll: Array,
});

const emit = defineEmits(["onSelectAll", "onCheckbox"]);

const checkbox = ref({
  model: {},
  isSelected: false,
});

const isFolder = computed(() => {
  return props.model.groupTreeNodes;
});

const handleCheckbox = (val) => {
  emit("onCheckbox", val);
};

const emitCheckbox = () => {
  console.log(checkbox.value);
  emit("onCheckbox", checkbox.value);
};

const selectAll = ref({
  model: {},
  isSelected: false,
});

const selectAllFunc = () => {
  // console.log("selectAll", selectAll.value);

  const items = selectAll.value.model?.groupTreeNodes;

  const checkIfNode = (node) => {
    // console.log("node", node);

    if (node && node.length) {
      for (let i of node) {
        if (!i.groupTreeNodes) {
          checkbox.value = {
            model: i,
            isSelected: true,
          };
          // console.log("onCheckbox", checkbox.value);
          emit("onCheckbox", checkbox.value);
        }
        if (Array.isArray(i.groupTreeNodes) && i.groupTreeNodes.length) {
          checkIfNode(i.groupTreeNodes);
        }
      }
    }
  };

  if (selectAll.value.isSelected) {
    checkIfNode(items);
  }
};

// watch(
//   () => checkbox.value,
//   () => {
//     console.log('watch',checkbox.value)
//   },{immediate: true}
// );

watchEffect(()=>{
  console.log('watch',checkbox.value)
})

onMounted(() => {
  if (props.defaultCheckbox && props.defaultCheckbox.length) {
    for (let defaultId of props.defaultCheckbox) {
      if (props.model.groupId === defaultId) {
        checkbox.value = {
          model: props.model,
          isSelected: true,
        };
        // console.log("onmounted", checkbox.value);
        emit("onCheckbox", checkbox.value);
      }
    }
  }
});
</script>
<style scoped>
.v-list-group__items .v-list-item {
  padding-inline-start: calc(var(--indent-padding)) !important;
}
</style>
