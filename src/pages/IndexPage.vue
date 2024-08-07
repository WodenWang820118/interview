<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="handleAdd">新增</q-btn>
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

      <!-- Edit Dialog -->
      <q-dialog v-model="editDialog">
        <q-card>
          <q-card-section>
            <div class="text-h6">Edit Item</div>
          </q-card-section>

          <q-card-section>
            <q-input v-model="tempData.name" label="姓名" />
            <q-input v-model="tempData.age" label="年齡" />
          </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="Cancel" v-close-popup />
            <q-btn flat label="Save" color="primary" @click="saveEdit" />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref } from 'vue';
import { useQuasar } from 'quasar';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface Item {
  name: string;
  age: number;
}

const $q = useQuasar();
const blockData = ref([
  {
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

const tempData = ref({
  name: '',
  age: 0,
});

const editDialog = ref(false);
let editingItemIndex = ref(-1);

function handleClickOption(btn: btnType, data: Item) {
  switch (btn.status) {
    case 'edit':
      // if desired to make editted data only available in the dialog
      // we might use a separate tempData for edit
      tempData.value = { ...data };
      editingItemIndex.value = blockData.value.findIndex(
        (item) => item.name === data.name && item.age === data.age
      );
      editDialog.value = true;
      break;
    case 'delete':
      if (confirm('Are you sure you want to delete this item?')) {
        const index = blockData.value.findIndex(
          (item) => item.name === data.name && item.age === data.age
        );
        if (index !== -1) {
          blockData.value.splice(index, 1);
          $q.notify({
            type: 'positive',
            message: 'Item deleted successfully',
          });
        } else {
          $q.notify({
            type: 'negative',
            message: 'Error: Item not found',
          });
        }
      }
      break;
    default:
      console.warn('Unknown button status:', btn.status);
  }
}

function handleAdd() {
  if (tempData.value.name && tempData.value.age) {
    blockData.value.push({ ...tempData.value });
    tempData.value = { name: '', age: 0 };
    $q.notify({
      type: 'positive',
      message: 'Item added successfully',
    });
  } else {
    $q.notify({
      type: 'negative',
      message: 'Please fill in all fields',
    });
  }
}

function saveEdit() {
  if (editingItemIndex.value !== -1) {
    blockData.value[editingItemIndex.value] = { ...tempData.value };
    $q.notify({
      type: 'positive',
      message: 'Item updated successfully',
    });
    editDialog.value = false;
    tempData.value = { name: '', age: 0 };
    editingItemIndex.value = -1;
  }
}
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
