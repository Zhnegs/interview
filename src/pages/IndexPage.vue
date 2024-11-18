<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => (val && val.length > 0) || '不得空白']"
        />
        <q-input
          v-model="tempData.age"
          label="年齡"
          type="number"
          :rules="[(val) => (val && val > 0) || '不得空白']"
        />
        <q-btn
          color="primary"
          class="q-mt-md"
          @click="blueBtnClickOption(btnMode.status)"
          :disable="!isFormValid"
          >{{ btnMode.label }}</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, computed } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: string;
}
const apiUrl = 'https://dahua.metcfire.com.tw/api/CRUDTest';

const blockData = ref([
  {
    id: '1',
    name: 'test',
    age: 25,
  },
]);

const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);

const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const btnMode = ref({
  label: '新增',
  status: 'add',
});

const isFormValid = computed(() => {
  return tempData.value.name !== '' && tempData.value.age !== '';
});

const tempData = ref({
  name: '',
  age: '',
});

function handleClickOption(btn, data) {
  // ...
  if (btn.status == 'edit') {
    tempData.value = JSON.parse(JSON.stringify(data));
    btnMode.value.label = '更新';
    btnMode.value.status = 'edit';
  } else if (btn.status == 'delete') {
    if (confirm('是否確定刪除該筆資料?')) {
      deleteData(tempData.value.id);
    }
  }
}
function blueBtnClickOption(btn_mode) {
  // ...
  if (btn_mode == 'add') {
    addData();
  } else if (btn_mode == 'edit') {
    updateData();
  }
  btnMode.value.label = '新增';
  btnMode.value.status = 'add';
  tempData.value = {
    name: '',
    age: '',
  };
}
async function getData() {
  try {
    const response = await axios.get(`${apiUrl}/a`);
    blockData.value = response.data;
  } catch (error) {
    console.error('getData()Error:', error);
  }
}
async function addData() {
  try {
    const response = await axios.post(apiUrl, tempData.value);
    blockData.value.push(response.data);
  } catch (error) {
    console.error('addData()Error:', error);
  }
}
async function updateData() {
  try {
    const response = await axios.put(apiUrl, tempData.value);
    const index = blockData.value.findIndex(
      (item) => item.id === tempData.value.id
    );
    if (index !== -1) blockData.value[index] = response.data;
  } catch (error) {
    console.error('updateData()Error:', error);
  }
}
async function deleteData(id: string) {
  try {
    await axios.delete(`${apiUrl}/${id}`);
    blockData.value = blockData.value.filter((item) => item.id !== id);
  } catch (error) {
    console.error('deleteData()Error:', error);
  }
}

//getData();
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
