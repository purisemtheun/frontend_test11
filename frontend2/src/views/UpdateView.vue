<template>
    <div class="q-gutter-md" style="max-width:400px;">
      <q-form @submit.prevent="onSubmit" class="q-gutter-md">
        <q-input v-model="id" label="ID" readonly />
        <q-input v-model="first_name" label="ชื่อ" />
        <q-input v-model="last_name" label="นามสกุล" />
        <q-input v-model="age" label="อายุ" type="number" />
        <q-input v-model="gender" label="เพศ" type="text" />  
        <q-input v-model="interests" label="สิ่งที่สนใจ" type="text" />  
        <q-input v-model="description" label="คำอธิบาย" type="text" />  
  
        <q-btn type="submit" label="Update" color="primary" />
      </q-form>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from "vue";
  import { useRoute, useRouter } from "vue-router";
  
  const route = useRoute();
  const router = useRouter();
  
  const id = ref(route.params.id);
  const first_name = ref("");
  const last_name = ref("");
  const age = ref("");
  const gender = ref("");
  const interests = ref("");  
  const description = ref(""); 
  
  const fetchData = async () => {
    try {
      const response = await fetch(`http://localhost:8000/users/${id.value}`);
      if (!response.ok) throw new Error("Failed to fetch data");
  
      const result = await response.json();
      first_name.value = result.first_name || "";
      last_name.value = result.last_name || "";
      age.value = result.age || "";
      gender.value = result.gender || "";
      interests.value = result.interests || "";
      description.value = result.description || "";
    } catch (error) {
      console.error("Error fetching user:", error);
    }
  };
  
  onMounted(fetchData);
  
  const onSubmit = async () => {
    const rawData = {
      first_name: first_name.value,
      last_name: last_name.value,
      age: Number(age.value),
      gender: gender.value,
      interests: interests.value,
      description: description.value
    };
  
    console.log("Sending data to API:", rawData);
  
    try {
      const response = await fetch(`http://localhost:8000/users/${id.value}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(rawData)
      });
  
      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message || "Something went wrong!");
      }
  
      const result = await response.json();
      alert(result.message);
      console.log(result);
      if (result.status === "ok") {
        router.push("/");
      }
    } catch (error) {
      console.error("Error updating user:", error);
      alert(`Error: ${error.message}`);
    }
  };
  </script>
  