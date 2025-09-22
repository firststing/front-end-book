<template>
  <div class="bg-gray-50 min-h-screen font-sans">
    <main class="container mx-auto px-6 py-8">
      <div class="flex justify-between items-center mb-6">
        <h2 class="text-xl font-semibold text-gray-700">
          <i class="pi pi-book mr-2"></i>รายการหนังสือทั้งหมด
          <span class="text-gray-500 font-normal ml-2">3 เล่ม</span>
        </h2>
        <button
          class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg flex items-center"
        >
          <i class="pi pi-plus mr-2"></i>
          <span>เพิ่มหนังสือใหม่</span>
        </button>
      </div>

      <div class="bg-white p-4 rounded-lg shadow-sm mb-8 flex items-center space-x-4">
        <div class="relative flex-grow">
          <span class="absolute inset-y-0 left-0 flex items-center pl-3">
            <i class="pi pi-search text-gray-400"></i>
          </span>
          <input
            type="text"
            placeholder="ค้นหาชื่อหนังสือ, ชื่อผู้แต่ง หรือ ISBN..."
            class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
        <select
          class="border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option>ทั้งหมด</option>
        </select>
        <select
          class="border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option>ทั้งหมด</option>
        </select>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        <div
          v-for="book in books"
          :key="book.isbn"
          class="bg-white rounded-lg shadow-md overflow-hidden transform hover:-translate-y-1 transition-transform duration-300"
        >
          <div class="p-6">
            <div class="flex justify-between items-start mb-2">
              <h3 class="text-lg font-bold text-gray-800">{{ book.title }}</h3>
              <span
                :class="[
                  'text-xs font-semibold px-3 py-1 rounded-full',
                  book.status.class,
                ]"
                >{{ book.status.text }}</span
              >
            </div>
            <p class="text-sm text-gray-600 mb-4">โดย {{ book.author }}</p>

            <div class="space-y-2 text-sm text-gray-700">
              <div class="flex">
                <span class="w-20 font-semibold">หมวดหมู่:</span>
                <span>{{ book.category }}</span>
              </div>
              <div class="flex">
                <span class="w-20 font-semibold">ปีที่ออก:</span>
                <span>{{ book.year }}</span>
              </div>
              <div class="flex">
                <span class="w-20 font-semibold">ISBN:</span>
                <span class="font-mono">{{ book.isbn }}</span>
              </div>
            </div>

            <p class="text-sm text-gray-600 mt-4 h-16">{{ book.description }}</p>

            <hr class="my-4" />

            <div class="flex justify-between items-center text-gray-600">
              <button class="hover:text-blue-600 flex items-center">
                <i class="pi pi-file-alt mr-2"></i>รายละเอียด
              </button>
              <div class="flex space-x-4">
                <button class="hover:text-green-600">
                  <i class="pi pi-pencil"></i>
                </button>
                <button class="hover:text-red-600">
                  <i class="pi pi-trash"></i>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref } from "vue";

// Dummy data for books based on the image
const books = ref([
  {
    title: "การเขียนโปรแกรม...",
    author: "นาย A. Developer",
    category: "เทคโนโลยี",
    year: 2023,
    isbn: "978-616-123-456-7",
    description:
      "หนังสือสำหรับเรียนรู้การพัฒนาเว็บไซต์และเว็บแอปพลิเคชันด้วย React แบบครบจบในเล่ม",
    status: {
      text: "พร้อมใช้งาน",
      class: "bg-green-100 text-green-800",
    },
  },
  {
    title: "ประวัติศาสตร์ไทยสมั...",
    author: "รศ.ดร. สมชาย วิชาการ",
    category: "ประวัติศาสตร์",
    year: 2022,
    isbn: "978-616-789-812-3",
    description: "บึกทึกเหตุการณ์สำคัญในประวัติศาสตร์ไทยตั้งแต่สมัยอดีตจนถึงปัจจุบัน",
    status: {
      text: "ไม่พร้อมแล้ว",
      class: "bg-yellow-100 text-yellow-800",
    },
  },
  {
    title: "นิยายรักในสวนดอกไม้",
    author: "คุณหญิง มายานี",
    category: "นิยาย",
    year: 2024,
    isbn: "978-616-456-789-0",
    description: "เรื่องราวความรักที่เกิดขึ้นในสวนดอกไม้อันสวยงาม",
    status: {
      text: "ของบำรุง",
      class: "bg-red-100 text-red-800",
    },
  },
]);

</script>

<style scoped>
/* PrimeIcons for icons */
@import url("https://unpkg.com/primeicons/primeicons.css");

/* Custom font, you can change it to your preferred Thai font */
body {
  font-family: 'Sarabun', sans-serif;
}
</style>