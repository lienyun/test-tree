<template lang="">
  <v-card>
    <v-card-title>選擇場域</v-card-title>
    <v-card-text>
      <v-list density="compact" nav class="pa-0 pl-3">
        <TestGroupTree
          :model="model"
          :allItems="itemsArr"
          :defaultCheckbox="defaultCheckbox"
          @onCheckbox="handleCheckbox"
        />
      </v-list>
    </v-card-text>
    <v-card-actions class="d-flex justify-end">
      <v-btn color="red" @click="emit('openDialog', false)">取消</v-btn>
      <v-btn color="success" @click="saveGroup">確定</v-btn>
    </v-card-actions>
  </v-card>
</template>
<script setup>
import { ref, onMounted, watch } from "vue";
import TestGroupTree from "./TestGroupTree.vue";

const props = defineProps({
  isDialog: Boolean,
  model: Object,
  defaultCheckbox: Array,
});

const selectAllArr = ref([]);

const selectAll = (val) => {
  console.log(val);
  selectAllArr.value = val;
  console.log("selectAllArr", selectAllArr.value);
};

const itemsArr = ref([]);

watch(
  () => props.model,
  () => {
    const items = props.model?.groupTreeNodes;

    const checkIfNode = (node) => {
      if (node && node.length) {
        for (let i of node) {
          if (!i.groupTreeNodes) {
            itemsArr.value.push(i);
          }
          if (Array.isArray(i.groupTreeNodes) && i.groupTreeNodes.length) {
            checkIfNode(i.groupTreeNodes);
          }
        }
      }
    };

    checkIfNode(items);
    console.log(itemsArr.value);
  },
  { immediate: true }
);

const selected = ref([])

const checkbox = ref([]);
const handleCheckbox = (val) => {
  console.log("handleCheckbox", val);
  if (val.isSelected) {
    checkbox.value.push(val.model);
  } else {
    checkbox.value = checkbox.value.filter(
      (i) => i.groupId !== val.model.groupId
    );
  }
  console.log("完成", checkbox.value);
};

const isSelectAll = ref([])

const emit = defineEmits(["onSelectedGroup", "openDialog"]);

const saveGroup = () => {
  let groupIds = [];
  let texts = [];

  if (checkbox.value.length) {
    for (let group of checkbox.value) {
      groupIds.push(group.groupId);
      texts.push(group.groupName);
    }
  }

  emit("getGroupId", groupIds);
  emit("getGroupText", texts);

  emit("openDialog", false);
};
</script>
<style lang=""></style>
