<template>
  <div class="q-gutter-md" style="max-width:400px;">
    <q-form @submit.prevent="onSubmit" class="q-gutter-md">
      <q-input v-model="formData.firstname" label="ชื่อ" />
      <q-input v-model="formData.lastname" label="นามสกุล" />
      <q-input v-model="formData.age" label="อายุ" type="number" />
      <q-input v-model="formData.gender" label="เพศ" />
      <q-input v-model="formData.interests" label="สิ่งที่สนใจ" />
      <q-input v-model="formData.description" label="คำอธิบาย" />
      <div v-if="errorMessage" class="text-negative q-mb-md">
        {{ errorMessage }}
      </div>
      <q-btn type="submit" label="Submit" color="primary" :loading="loading" />
    </q-form>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import router from '../router/index.js'

// แก้ไขชื่อฟิลด์ให้ตรงกับ backend ที่ใช้ firstname และ lastname (ไม่มี underscore)
const formData = reactive({
  firstname: '',
  lastname: '',
  age: '',
  gender: '',
  interests: '',
  description: ''
})

const loading = ref(false)
const errorMessage = ref('')

const onSubmit = async () => {
  errorMessage.value = ''
  
  // ตรวจสอบข้อมูลครบทุกช่อง
  if (!formData.firstname || !formData.lastname || !formData.age || 
      !formData.gender || !formData.interests || !formData.description) {
    errorMessage.value = 'กรุณากรอกข้อมูลให้ครบถ้วน'
    return
  }
  
  try {
    loading.value = true
    console.log("ส่งข้อมูล:", formData)
    
    const myHeaders = new Headers()
    myHeaders.append("Content-Type", "application/json")

    // ส่งข้อมูลโดยใช้ชื่อฟิลด์ที่ตรงกับ backend
    const raw = JSON.stringify({
      firstname: formData.firstname,
      lastname: formData.lastname,
      age: formData.age,
      gender: formData.gender,
      interests: formData.interests,
      description: formData.description
    })

    const requestOptions = {
      method: "POST",
      headers: myHeaders,
      body: raw,
      redirect: "follow"
    }

    const response = await fetch("http://localhost:8000/users", requestOptions)
    console.log("Status:", response.status)
    
    const responseText = await response.text()
    console.log("Response text:", responseText)
    
    if (responseText) {
      try {
        const result = JSON.parse(responseText)
        console.log("Parsed result:", result)
        
        if (result.message === 'insert ok') {
          alert('บันทึกข้อมูลสำเร็จ')
          router.push('/')
        } else {
          // แสดงข้อความผิดพลาดจาก backend ถ้ามี
          errorMessage.value = result.message || 'เกิดข้อผิดพลาดจากเซิร์ฟเวอร์'
          
          // แสดงรายการข้อผิดพลาดถ้ามี
          if (result.errors && result.errors.length > 0) {
            errorMessage.value += ': ' + result.errors.join(', ')
          }
        }
      } catch (parseError) {
        console.error("JSON parse error:", parseError)
        errorMessage.value = 'เกิดข้อผิดพลาดในการอ่านข้อมูลจากเซิร์ฟเวอร์'
      }
    } else {
      errorMessage.value = 'ไม่ได้รับข้อมูลตอบกลับจากเซิร์ฟเวอร์'
    }
  } catch (error) {
    console.error("Network error:", error)
    errorMessage.value = `เกิดข้อผิดพลาดในการเชื่อมต่อ: ${error.message}`
  } finally {
    loading.value = false
  }
}
</script>