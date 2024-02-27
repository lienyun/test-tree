<template lang="">
  <v-card class="pa-5">
    <v-row>
      <v-col cols="12" md="3">
        <v-dialog v-model="dialog" max-width="500" scrollable>
          <template v-slot:activator="{ props }">
            <v-select
              v-model="selected.groupText"
              label="選擇場域*"
              variant="outlined"
              density="compact"
              @focus="treeStore.getGroup(); dialog = true"
              hide-no-data
            >
              <template #selection="{ item, index }">
                <v-chip v-if="index < 1">
                  <span>{{ item.title }}</span>
                </v-chip>

                <span
                  v-if="index === 1"
                  class="text-grey text-caption align-self-center"
                >
                  (+{{ selected.groupId.length - 1 }} others)
                </span>
              </template>
            </v-select>
          </template>
          
          <TestGroupTreeCard
            :model="treeStore.groupObj"
            :defaultCheckbox="selected.groupId"
            @getGroupId="getGroupId"
            @getGroupText="getGroupText"
            @openDialog="isOpen"
          />
        </v-dialog>
      </v-col>
      <v-col cols="12" md="3">
        <v-select
          v-model="selected.deviceCategory"
          :items="optionArr.category"
          item-title="text"
          item-value="value"
          label="裝置類別"
          variant="outlined"
          density="compact"
        ></v-select>
      </v-col>
      <v-col cols="12" md="3">
        <v-text-field
          v-model="selected.deviceName"
          label="裝置名稱"
          variant="outlined"
          density="compact"
        ></v-text-field>
      </v-col>
      <v-col cols="12" md="3">
        <v-select
          v-model="selected.connStatus"
          :items="optionArr.connStatus"
          item-title="text"
          item-value="value"
          label="連線狀態"
          variant="outlined"
          density="compact"
          clearable
        ></v-select>
      </v-col>
      <v-col cols="12" md="3">
        <v-text-field
          v-model="selected.deviceId"
          label="裝置ID"
          variant="outlined"
          density="compact"
        ></v-text-field>
      </v-col>
      <v-spacer></v-spacer>
      <v-col cols="12" md="3" class="d-flex justify-end align-center">
        <v-btn
          prepend-icon="mdi-window-close"
          color="red"
          rounded="default"
          @click="clearSelect"
          >清除</v-btn
        >
        <v-btn
          prepend-icon="mdi-magnify"
          color="success"
          rounded="default"
          class="ml-5"
          @click="getData"
          >搜尋</v-btn
        >
      </v-col>
    </v-row>
  </v-card>

  <v-card class="mt-5">
    <v-card-title class="d-flex justify-space-between align-end"
      ><div>查詢結果<br />裝置列表</div>
      <v-btn prepend-icon="mdi-filter" rounded="default">
        選擇欄位
        <v-menu
          activator="parent"
          location="start"
          :close-on-content-click="false"
        >
          <v-list>
            <v-list-item>
              <v-checkbox
                v-for="(item, index) in tableCheckBox"
                :key="index"
                :value="item.value"
                density="compact"
                :label="item.text"
                hide-details
                v-model="tableSelected"
              >
              </v-checkbox>
            </v-list-item>
          </v-list>
        </v-menu>
      </v-btn>
    </v-card-title>
    <v-card-text>
      <EasyDataTable
        :headers="headers"
        :items="items"
        :table-class-name="darkStore.tableMode"
        no-hover
        id="datatable"
      >
        <template #item-connStatus="{ connStatus }">
          <v-chip
            :class="Number(connStatus) ? 'bg-success' : 'bg-warning'"
            rounded
            density="comfortable"
            >{{
              Number(connStatus) ? t("isConnect", 1) : t("isConnect", 2)
            }}</v-chip
          >
        </template>

        <template #item-manage="{ deviceId, deviceName, deviceCategory }">
          <v-tooltip text="編輯資訊" location="bottom">
            <template v-slot:activator="{ props }">
              <router-link :to="`/ec2503/edit-${deviceCategory}/${deviceId}`">
                <v-btn
                  icon="mdi-pencil"
                  variant="text"
                  size="x-small"
                  v-bind="props"
                  color="linkColor"
                ></v-btn>
              </router-link>
            </template>
          </v-tooltip>

          <v-tooltip text="事件設定" location="bottom">
            <template v-slot:activator="{ props }">
              <router-link :to="`/ec2503/event/${deviceId}`">
                <v-btn
                  icon="mdi-file-document-edit"
                  variant="text"
                  size="x-small"
                  v-bind="props"
                  color="linkColor"
                ></v-btn>
              </router-link>
            </template>
          </v-tooltip>

          <v-tooltip text="備註" location="bottom">
            <template v-slot:activator="{ props }">
              <v-btn
                icon="mdi-note"
                variant="text"
                size="x-small"
                v-bind="props"
                color="linkColor"
                @click="openNote(deviceId, deviceCategory)"
              ></v-btn>
            </template>
          </v-tooltip>
          <!-- <v-dialog width="30%">
            <template v-slot:activator="{ props: dialog }">
              <v-tooltip text="刪除" location="bottom">
                <template v-slot:activator="{ props: tooltip }">
                  <v-btn
                    v-bind="mergeProps(dialog, tooltip)"
                    icon="mdi-delete"
                    variant="text"
                    size="x-small"
                    color="linkColor"
                  ></v-btn>
                </template>
              </v-tooltip>
            </template>

            <template v-slot:default="{ isActive }">
              <v-card title="提示">
                <v-card-text>
                  確定要刪除「{{
                    deviceName
                  }}」？刪除時間將依照資料多寡而不同，操作時請勿關閉瀏覽器，並耐心等候。
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>

                  <div class="text-center">
                    <router-link to="/ec2503">
                      <v-btn
                        class="mt-5"
                        color="red"
                        variant="text"
                        @click="isActive.value = false"
                        >關閉</v-btn
                      >
                    </router-link>
                    <router-link to="/ec2503">
                      <v-btn
                        class="mt-5"
                        color="success"
                        variant="text"
                        @click="isActive.value = false"
                        >確認</v-btn
                      >
                    </router-link>
                  </div>
                </v-card-actions>
              </v-card>
            </template>
          </v-dialog> -->
        </template>
      </EasyDataTable>
    </v-card-text>
  </v-card>

  <v-dialog v-model="note.dialog" width="500">
    <v-card title="備註">
      <v-card-text>
        <v-text-field
          label="內容"
          variant="outlined"
          density="compact"
          hide-details
          v-model="note.content"
        ></v-text-field>

        <v-file-input
          label="上傳圖片(格式:.tiff, .jpg, .bmp, .jpeg)"
          variant="outlined"
          hide-details
          density="compact"
          class="mt-5"
          prepend-icon="mdi-image-plus"
          chips
          accept="image/png, image/jpeg, image/bmp, image/tiff"
          v-model="note.images.file"
          multiple
          @change="onSelectFile"
          clearable
          @click:clear="clearPreview"
        ></v-file-input>

        <v-carousel
          v-if="note.images.length"
          class="mt-5"
          hide-delimiters
          height="250"
        >
          <template v-slot:prev="{ props }">
            <v-btn
              variant="elevated"
              color="surface"
              @click="props.onClick"
              icon="mdi-chevron-left"
            ></v-btn>
          </template>
          <template v-slot:next="{ props }">
            <v-btn
              variant="elevated"
              color="surface"
              @click="props.onClick"
              icon="mdi-chevron-right"
            ></v-btn>
          </template>
          <v-carousel-item
            v-for="image in note.images"
            :src="image.url"
            cover
            rounded
          ></v-carousel-item>
        </v-carousel>
      </v-card-text>
      <v-card-actions class="d-flex justify-end">
        <v-btn color="red" variant="text" @click="note.dialog = false"
          >取消</v-btn
        >

        <v-btn color="success" variant="text" @click="updateNote">儲存</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  <SnackBar :snackbar="snackbar" />
</template>
<script setup>
import { ref, watch, reactive, mergeProps, computed, watchEffect } from "vue";
import { useRouter, useRoute } from "vue-router";

import TestGroupTreeCard from "../../components/Global/Tree/TestGroupTreeCard.vue";

import SnackBar from "../../components/Global/SnackBar.vue";

import { useDarkStore } from "../../stores/darkMode";
const darkStore = useDarkStore();

import { useGetTreeStore } from "../../stores/getTree";
const treeStore = useGetTreeStore();

import { useI18n } from "vue-i18n";

import tw from "../../language/deviceTypeList/zh-TW.json";
import en from "../../language/deviceTypeList/en-US.json";
import cn from "../../language/deviceTypeList/zh-CN.json";

import ExportMenu from "../../components/Global/ExportMenu.vue";
import axios from "axios";


const $router = useRouter();
const $route = useRoute();

$router.beforeEach((to, from) => {
  if (from.name === "資訊編輯") {
    getData();
  }
});

const { t } = useI18n({
  locale: "zh-TW",
  fallbackLocale: "en",
  messages: {
    "zh-TW": tw,
    "zh-CN": cn,
    "en-US": en,
  },
});

const selected = ref({
  groupId: [],
  groupText: ["全部"],
  deviceName: null,
  deviceCategory: [],
  deviceId: null,
  connStatus: null,
});

const hintText = ref("請填入任意條件");

//datatable
const headers = ref([
  { text: "隸屬場域", value: "groupname", sortable: true },
  { text: "裝置名稱", value: "deviceName", sortable: true },
  { text: "裝置ID", value: "deviceId", sortable: true },
  { text: "經度", value: "lng", sortable: true },
  { text: "緯度", value: "lat", sortable: true },
  { text: "裝置型號", value: "deviceType", sortable: true },
  { text: "連線狀態", value: "connStatus", sortable: true },
  { text: "表計模式", value: "connStatus", sortable: true },
  { text: "管理", value: "manage" },
]);

//呈現在datatable的資料
const items = ref([]);
const dialog = ref(false);

const isOpen = (val) => {
  dialog.value = val;
};


const getGroupId = (val) => {
  selected.value.groupId = val;
};

const getGroupText = (val) => {
  selected.value.groupText = val;
};

const optionArr = ref({
  category: [
    { text: "電表", value: "powerMeter" },
    { text: "水表", value: "waterMeter" },
    { text: "水位計", value: "waterGauge" },
    { text: "閘道器", value: "gateway" },
  ],
  connStatus: [
    { text: "連線", value: "1" },
    { text: "斷線", value: "0" },
  ],
});

const tableCheckBox = ref([
  { text: "隸屬場域", value: "groupname" },
  { text: "裝置名稱", value: "deviceName" },
  { text: "裝置ID", value: "deviceId" },
  { text: "經度", value: "lng" },
  { text: "緯度", value: "lat" },
  { text: "裝置型號", value: "deviceType" },
  { text: "連線狀態", value: "connStatus" },
  { text: "表計模式", value: "type" },
  { text: "高低壓電號", value: "powerNum" },
  { text: "管理", value: "manage" },
]);
// 預設勾選 需再加入 表計模式 高低壓電號
const tableSelected = ref([
  "groupname",
  "deviceName",
  "deviceId",
  "lng",
  "lat",
  "deviceType",
  "connStatus",
  "manage",
]);

// table header勾選選項
watch(
  () => tableSelected.value,
  (newValue, oldValue) => {
    headers.value = tableCheckBox.value.filter((title) => {
      return newValue.includes(title.value);
    });
  },
  { immediate: true }
);

watch(
  () => selected.value.deviceCategory,
  (newValue, oldValue) => {
    tableCheckBox.value = [
      { text: "隸屬場域", value: "groupname", sortable: true },
      { text: "裝置名稱", value: "deviceName", sortable: true },
      { text: "裝置ID", value: "deviceId", sortable: true },
      { text: "經度", value: "lng", sortable: true },
      { text: "緯度", value: "lat" },
      { text: "裝置型號", value: "deviceType", sortable: true },
      { text: "連線狀態", value: "connStatus", sortable: true },
      { text: "管理", value: "manage" },
    ];

    if (newValue === "powerMeter") {
      tableCheckBox.value.splice(-1, 0, {
        text: "表計模式",
        value: "type",
        sortable: true,
      });
      tableCheckBox.value.splice(-1, 0, {
        text: "高低壓電號",
        value: "powerNum",
        sortable: true,
      });
    } else if (newValue === "waterMeter") {
      tableCheckBox.value.splice(-1, 0, {
        text: "表計模式",
        value: "type",
        sortable: true,
      });
    } else if (newValue === "waterGauge") {
      tableCheckBox.value.splice(-1, 0, {
        text: "當日水位",
        value: "waterLevel",
        sortable: true,
      });
    }
  },
  { immediate: true }
);

// 備註
const note = ref({
  dialog: false,
  content: null,
  images: [],
});

// image preview
const onSelectFile = (e) => {
  const files = e.target.files;
  for (let i = 0; i < files.length; i++) {
    const reader = new FileReader();
    reader.onload = (e) => {
      note.value.images.push({
        file: files[i],
        url: e.target.result,
      });
    };
    reader.readAsDataURL(files[i]);
  }
};

const clearPreview = async () => {
  note.value.images = [];
};

const getData = async () => {
  // 重新設定篩選條件
  console.log(selected.value);
  const setNull = (value) => (value ? value : "");

  let { deviceCategory, deviceName, deviceId, connStatus } = selected.value;

  deviceCategory = setNull(deviceCategory);
  deviceName = setNull(deviceName);
  deviceId = setNull(deviceId);
  connStatus = setNull(connStatus);
  let groupId;
  if (selected.value.groupId) {
    groupId = selected.value.groupId;
  } else {
    groupId = "";
  }
  console.log(groupId);
  const url = `${
    import.meta.env.VITE_API_URL
  }infactory-backend/api/devicekanban/v2/getklport?categories=${deviceCategory}&groupids=${groupId}&devicename=${deviceName}&connectionstatus=${connStatus}&deviceid=${deviceId}`;

  await axios
    .get(url)
    .then((res) => {
      console.log(res.data.data);
      if (res.data.data.length) {
        items.value = res.data.data.map((item) => ({
          ...item,
          deviceType: t(item.deviceType),
        }));
        // 其他屬性待添加
        // let newData = res.data.data.map((item) => {
        //   if (selected.value.deviceCategory === "powerMeter") {
        //     console.log(item.attrs);
        //   }
        // });
      } else {
        items.value = [];
        hintText.value = "查無資料，請重新查詢";
      }
    })
    .catch((err) => console.log(err));

  tableSelected.value = [
    "groupname",
    "deviceName",
    "deviceId",
    "lng",
    "lat",
    "deviceType",
    "connStatus",
    "manage",
  ];
};

await getData();

const clickNote = ref({
  deviceId: null,
  deviceCategory: null,
});

const openNote = async (deviceId, deviceCategory) => {
  clickNote.value.deviceId = deviceId;
  clickNote.value.deviceCategory = deviceCategory;

  note.value.dialog = true;
  const url = `${
    import.meta.env.VITE_API_URL
  }infactory-backend/api/devicekanban/v2/getklportedit?deviceId=${deviceId}&categories=${deviceCategory}`;
  const res = await axios.get(url);
  const content = res.data.data.attrIdVo.filter((i) => i.attrId === 501000600);
  note.value.content = content[0].value;
  const image = res.data.data.attrIdVo.filter((i) => i.attrId === 603601);
};

const snackbar = ref({
  show: false,
  icon: "mdi-close-circle",
  text: "",
  color: "red",
});

watchEffect(() => {
  if ($route.meta.snackbar) {
    snackbar.value = $route.meta.snackbar;
    $route.meta.snackbar = null;
  }
});
const updateNote = async () => {
  const url = `${
    import.meta.env.VITE_API_URL
  }infactory-backend/api/devicekanban/v2/updateklport?deviceId=${
    clickNote.value.deviceId
  }&categories=${clickNote.value.deviceCategory}`;
  const putData = {
    attributes: [
      {
        attrId: "501000600",
        value: note.value.content,
      },
      {
        attrId: "603601",
        value: "test null",
      },
    ],
  };
  console.log(putData);
  await axios
    .put(url, putData)
    .then((res) => {
      console.log("成功", res);
      note.value.dialog = false;
      snackbar.value = {
        show: true,
        icon: "mdi-check-circle",
        text: "增加備註成功",
        color: "success",
      };
      note.value = {
        dialog: false,
        content: null,
        images: [],
      };
    })
    .catch((err) => {
      console.log("失敗", err);
      snackbar.value = {
        show: true,
        icon: "mdi-close-circle",
        text: "增加備註失敗",
        color: "red",
      };
    });
};

const clearSelect = () => {
  selected.value = {
    groupId: [],
    groupText: [],
    deviceName: null,
    deviceCategory: [],
    deviceId: null,
    connStatus: null,
  };
  items.value = [];
  tableCheckBox.value = [
    { text: "隸屬場域", value: "groupname" },
    { text: "裝置名稱", value: "deviceName" },
    { text: "裝置ID", value: "deviceId" },
    { text: "經度", value: "lng" },
    { text: "緯度", value: "lat" },
    { text: "裝置型號", value: "deviceType" },
    { text: "連線狀態", value: "connStatus" },
  ];
  hintText.value = "請填入任意條件";
};
</script>
<style scoped></style>
