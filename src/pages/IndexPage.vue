<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="user.name" label="姓名" />
        <q-input v-model="user.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="handleAdd">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="users"
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
              <div :contenteditable="editable">{{ col.value }}</div>
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
import { QTableProps, useQuasar } from 'quasar';
import { ref } from 'vue';
const $q = useQuasar();
const editable = ref(false);
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const user = ref({
  name: '',
  age: '',
});

const users = ref([]);
const addUser = () => {
  users.value.push({ ...user.value, id: new Date().getTime().toString() });
};
const removeUser = (id) => {
  users.value.splice(
    users.value.findIndex((u) => u.id === id),
    1
  );
};

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

const handleAdd = () => {
  addUser();
  // axios
  axios
    .post('', user.value)
    .then(function (response) {
      // id 應由 server side 產生
      users.value.push(response.data);
      user.value = { name: '', age: '' };
      $q.notify('註冊成功');
    })
    .catch(function (error) {
      $q.notify(error.message);
    });
};

function handleClickOption(btn, data) {
  // ...

  if (btn.status === 'delete') {
    removeUser(data.id);

    axios.delete('', {
      params: {
        id: data.id,
      },
    });
  }
  if (btn.status === 'edit') {
    editable.value = !editable.value;
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
