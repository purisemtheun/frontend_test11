<template>
  <div class="q-pa-md">
    <q-btn icon="add" @click="onCreate" />

    <!-- การเลือกกรองเพศ -->
    <q-select
  v-model="genderFilter"
  :options="genderOptions"
  label="กรองตามเพศ"
  class="q-mb-md"
  @input="filterData"
/>

    <q-table
      title="ค้นหาชื่อข้อมูลผู้ใช้"
      :rows="filteredRows"
      :columns="columns"
      row-key="id"
      :sort-methods="sortMethods"
    >
      <template v-slot:body-cell-actions="{ row }">
        <q-td>
          <q-btn
            flat
            dense
            round
            icon="edit"
            color="primary"
            @click="onEdit(row.id)"
          />
          <q-btn
            flat
            dense
            round
            icon="delete"
            color="red"
            @click="onDelete(row.id)"
          />
        </q-td>
      </template>
    </q-table>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import router from "../router/index.js";

const columns = [
  { name: "id", align: "center", label: "ไอดี", field: "id", sortable: true },
  { name: "firstname", align: "center", label: "ชื่อ", field: "firstname", sortable: true },
  { name: "lastname", align: "center", label: "นามสกุล", field: "lastname", sortable: true },
  { name: "age", align: "center", label: "อายุ", field: "age", sortable: true },
  { name: "gender", align: "center", label: "เพศ", field: "gender", sortable: true },
  { name: "interests", align: "center", label: "สิ่งที่สนใจ", field: "interests" },
  { name: "description", align: "center", label: "คำอธิบาย", field: "description" },
  { name: "actions", align: "center", label: "Actions" }
];

const rows = ref([]);
const filteredRows = ref([]); 
const genderFilter = ref(""); 

const genderOptions = [
  { label: "ชาย", value: "ชาย" },
  { label: "หญิง", value: "หญิง" },
  { label: "ไม่ระบุ", value: "ไม่ระบุ" }
];

const fetchData = async () => {
  try {
    const response = await fetch("http://localhost:8000/users");
    const result = await response.json();
    rows.value = result;
    filteredRows.value = result; // กรองเริ่มต้นแสดงทั้งหมด
  } catch (error) {
    console.error("Fetch Error:", error);
  }
};

fetchData();

const filterData = () => {
  if (!genderFilter.value || genderFilter.value === "") {
   
    filteredRows.value = rows.value;
  } else {
    
    filteredRows.value = rows.value.filter((row) => row.gender === genderFilter.value);
  }
};

const sortMethods = {
  firstname: (a, b) => a.firstname.localeCompare(b.firstname),
  lastname: (a, b) => a.lastname.localeCompare(b.lastname),
  age: (a, b) => a.age - b.age,
};

const onCreate = () => {
  router.push("/create");
};

const onEdit = (id) => {
  router.push("/update/" + id);
};

const onDelete = async (id) => {
  try {
    const response = await fetch(`http://localhost:8000/users/${id}`, { method: "DELETE" });
    const result = await response.json();
    alert(result.message);
    if (result.status === "ok") {
      fetchData();
    }
  } catch (error) {
    console.error("Delete Error:", error);
  }
};
</script>
