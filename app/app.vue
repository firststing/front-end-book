<script setup>
import { ref } from "vue";

// Login API
const { data: tokenRes, error: loginError } = await useFetch("http://localhost:3000/auth/login", {
  method: "post",
  body: {
    email: "firsttest@gmail.com",
    password: "first1234",
  },
});

// token จะอยู่ใน tokenRes.value
console.log("Login response:", tokenRes.value);

const token = tokenRes.value?.access_token || "";

// Books API
const { data: books, pending, error, refresh } = await useFetch("http://localhost:3000/books", {
  method: "get",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
});

// Modal state
const showAddModal = ref(false);
const showEditModal = ref(false);
const showDeleteModal = ref(false);
const isSubmitting = ref(false);

// Form data
const newBook = ref({
  tille: "",
  author: "",
  published_year: new Date().getFullYear(),
  genre: ""
});

const editBook = ref({
  id: null,
  tille: "",
  author: "",
  published_year: new Date().getFullYear(),
  genre: ""
});

const deleteBook = ref({
  id: null,
  tille: ""
});

// Add book function
const addBook = async () => {
  if (!newBook.value.tille || !newBook.value.author || !newBook.value.genre) {
    alert("กรุณากรอกข้อมูลให้ครบถ้วน");
    return;
  }

  if (!token) {
    alert("ไม่พบ Token กรุณา Login ใหม่");
    return;
  }

  isSubmitting.value = true;

  try {
    console.log("Adding book with token:", token);
    console.log("Book data:", newBook.value);
    
    const response = await fetch("http://localhost:3000/books", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${token}`,
        "Accept": "application/json",
      },
      body: JSON.stringify({
        tille: newBook.value.tille,
        author: newBook.value.author,
        published_year: newBook.value.published_year,
        genre: newBook.value.genre
      })
    });

    console.log("Response status:", response.status);
    console.log("Response headers:", Object.fromEntries(response.headers.entries()));

    if (!response.ok) {
      const errorText = await response.text();
      console.error("Error response:", errorText);
      
      // Handle specific NestJS errors
      let errorMessage = errorText;
      try {
        const errorJson = JSON.parse(errorText);
        errorMessage = errorJson.message || errorJson.error || errorText;
        if (Array.isArray(errorMessage)) {
          errorMessage = errorMessage.join(', ');
        }
      } catch (e) {
        // Use text as is
      }
      
      throw new Error(`HTTP ${response.status}: ${errorMessage}`);
    }

    const result = await response.json();
    console.log("Book added successfully:", result);
    
    // Reset form
    newBook.value = {
      tille: "",
      author: "",
      published_year: new Date().getFullYear(),
      genre: ""
    };
    // Close modal
    showAddModal.value = false;
    // Refresh books list
    await refresh();
    alert("เพิ่มหนังสือสำเร็จ!");

  } catch (error) {
    console.error("Error adding book:", error);
    
    // Handle network errors
    if (error.message === 'Failed to fetch') {
      alert("ไม่สามารถเชื่อมต่อกับเซิร์ฟเวอร์ได้ กรุณาตรวจสอบ:\n1. เซิร์ฟเวอร์ทำงานอยู่หรือไม่\n2. CORS configuration\n3. Network connection");
    } else {
      alert(`เกิดข้อผิดพลาดในการเพิ่มหนังสือ: ${error.message || error}`);
    }
  } finally {
    isSubmitting.value = false;
  }
};

// Edit book function
const openEditModal = (book) => {
  editBook.value = {
    id: book._id,
    tille: book.title || book.tille,
    author: book.author,
    published_year: book.published_year,
    genre: book.genre
  };
  showEditModal.value = true;
};

const updateBook = async () => {
  if (!editBook.value.tille || !editBook.value.author || !editBook.value.genre) {
    alert("กรุณากรอกข้อมูลให้ครบถ้วน");
    return;
  }

  if (!token) {
    alert("ไม่พบ Token กรุณา Login ใหม่");
    return;
  }

  isSubmitting.value = true;

  try {
    console.log("Updating book with ID:", editBook.value.id);
    console.log("Update data:", editBook.value);
    
    const response = await fetch(`http://localhost:3000/books/${editBook.value.id}`, {
      method: "PATCH",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${token}`,
        "Accept": "application/json",
      },
      mode: "cors",
      credentials: "include",
      body: JSON.stringify({
        tille: editBook.value.tille,
        author: editBook.value.author,
        published_year: editBook.value.published_year,
        genre: editBook.value.genre
      })
    });

    console.log("Response status:", response.status);

    if (!response.ok) {
      const errorText = await response.text();
      console.error("Error response:", errorText);
      throw new Error(`HTTP ${response.status}: ${errorText}`);
    }

    const result = await response.json();
    console.log("Book updated successfully:", result);
    
    // Close modal
    showEditModal.value = false;
    // Refresh books list
    await refresh();
    alert("แก้ไขหนังสือสำเร็จ!");

  } catch (error) {
    console.error("Error updating book:", error);
    alert(`เกิดข้อผิดพลาดในการแก้ไขหนังสือ: ${error.message || error}`);
  } finally {
    isSubmitting.value = false;
  }
};

// Delete book function
const openDeleteModal = (book) => {
  deleteBook.value = {
    id: book._id,
    tille: book.title || book.tille
  };
  showDeleteModal.value = true;
};

const confirmDelete = async () => {
  if (!token) {
    alert("ไม่พบ Token กรุณา Login ใหม่");
    return;
  }

  isSubmitting.value = true;

  try {
    console.log("Deleting book with ID:", deleteBook.value.id);
    
    const response = await fetch(`http://localhost:3000/books/${deleteBook.value.id}`, {
      method: "DELETE",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${token}`,
        "Accept": "application/json",
      },
      mode: "cors",
      credentials: "include"
    });

    console.log("Response status:", response.status);

    if (!response.ok) {
      const errorText = await response.text();
      console.error("Error response:", errorText);
      throw new Error(`HTTP ${response.status}: ${errorText}`);
    }

    // For DELETE, response might be empty
    let result;
    try {
      result = await response.json();
    } catch (e) {
      result = "Deleted successfully";
    }
    
    console.log("Book deleted successfully:", result);
    
    // Close modal
    showDeleteModal.value = false;
    // Refresh books list
    await refresh();
    alert("ลบหนังสือสำเร็จ!");

  } catch (error) {
    console.error("Error deleting book:", error);
    alert(`เกิดข้อผิดพลาดในการลบหนังสือ: ${error.message || error}`);
  } finally {
    isSubmitting.value = false;
  }
};

// Close modal
const closeModal = () => {
  showAddModal.value = false;
  showEditModal.value = false;
  showDeleteModal.value = false;
  // Reset forms
  newBook.value = {
    tille: "",
    author: "",
    published_year: new Date().getFullYear(),
    genre: ""
  };
  editBook.value = {
    id: null,
    tille: "",
    author: "",
    published_year: new Date().getFullYear(),
    genre: ""
  };
  deleteBook.value = {
    id: null,
    tille: ""
  };
};
</script>

<template>
  <div class="min-h-screen bg-gray-50">
    <main class="container mx-auto px-4 py-8">
      <!-- Header -->
      <div class="bg-white rounded-lg shadow-sm p-6 mb-8">
        <div class="flex items-center justify-between">
          <div class="flex items-center space-x-3">
            <div class="p-3 bg-blue-100 rounded-full">
              <i class="pi pi-book text-blue-600 text-xl"></i>
            </div>
            <div>
              <h1 class="text-2xl font-bold text-gray-900">รายการหนังสือทั้งหมด</h1>
              <p class="text-gray-600">
                <span class="font-medium text-blue-600">{{ books?.length || 0 }}</span> เล่ม
              </p>
            </div>
          </div>
          <button 
            @click="showAddModal = true"
            class="bg-blue-600 hover:bg-blue-700 text-white px-6 py-3 rounded-lg font-medium transition-colors duration-200 flex items-center space-x-2 shadow-sm"
          >
            <i class="pi pi-plus"></i>
            <span>เพิ่มหนังสือใหม่</span>
          </button>
        </div>
      </div>

      <!-- Loading / Error States -->
      <div v-if="pending" class="flex items-center justify-center py-16">
        <div class="text-center">
          <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
          <p class="text-gray-600">กำลังโหลดข้อมูล...</p>
        </div>
      </div>

      <div v-else-if="error" class="bg-red-50 border border-red-200 rounded-lg p-6 text-center">
        <div class="p-3 bg-red-100 rounded-full w-fit mx-auto mb-4">
          <i class="pi pi-exclamation-triangle text-red-600 text-2xl"></i>
        </div>
        <h3 class="text-red-800 font-medium mb-2">เกิดข้อผิดพลาด</h3>
        <p class="text-red-700">{{ error.message }}</p>
      </div>

      <!-- Books Grid -->
      <div v-else-if="books && books.length > 0">
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
          <div 
            v-for="book in books" 
            :key="book.isbn" 
            class="bg-white rounded-xl shadow-sm hover:shadow-md transition-shadow duration-200 overflow-hidden border border-gray-100"
          >
            <!-- Book Cover Placeholder -->
            <div class="h-48 bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center">
              <i class="pi pi-book text-4xl text-blue-400"></i>
            </div>
            
            <!-- Book Content -->
            <div class="p-5">
              <!-- Title -->
              <div class="mb-3">
                <h3 class="font-bold text-lg text-gray-900 line-clamp-2 mb-1">
                  {{ book.title || book.tille }}
                </h3>
                <p class="text-gray-600 text-sm">โดย {{ book.author }}</p>
              </div>

              <!-- Info -->
              <div class="space-y-2 mb-4">
                <div class="flex items-center justify-between text-sm">
                  <span class="text-gray-500">หมวดหมู่:</span>
                  <span class="px-2 py-1 bg-green-100 text-green-800 rounded-full text-xs font-medium">
                    {{ book.genre }}
                  </span>
                </div>
                <div class="flex items-center justify-between text-sm">
                  <span class="text-gray-500">ปีที่ออก:</span>
                  <span class="text-gray-900 font-medium">{{ book.published_year }}</span>
                </div>
              </div>

              <!-- Actions -->
              <div class="flex items-center justify-between pt-4 border-t border-gray-100">
                <button class="text-blue-600 hover:text-blue-700 font-medium text-sm transition-colors duration-200 flex items-center space-x-1">
                </button>
                
                <div class="flex items-center space-x-2">
                  <button 
                    @click="openEditModal(book)"
                    class="p-2 text-gray-400 hover:text-blue-600 hover:bg-blue-50 rounded-lg transition-colors duration-200"
                  >
                    <i class="pi pi-pencil text-sm"></i>
                  </button>
                  <button 
                    @click="openDeleteModal(book)"
                    class="p-2 text-gray-400 hover:text-red-600 hover:bg-red-50 rounded-lg transition-colors duration-200"
                  >
                    <i class="pi pi-trash text-sm"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-else class="text-center py-16">
        <div class="p-6 bg-gray-100 rounded-full w-fit mx-auto mb-4">
          <i class="pi pi-book text-4xl text-gray-400"></i>
        </div>
        <h3 class="text-xl font-medium text-gray-900 mb-2">ยังไม่มีหนังสือ</h3>
        <p class="text-gray-600 mb-6">เริ่มต้นสร้างคลังหนังสือของคุณ</p>
        <button 
          @click="showAddModal = true"
          class="bg-blue-600 hover:bg-blue-700 text-white px-6 py-3 rounded-lg font-medium transition-colors duration-200 inline-flex items-center space-x-2"
        >
          <i class="pi pi-plus"></i>
          <span>เพิ่มหนังสือแรก</span>
        </button>
      </div>
    </main>

    <!-- Add Book Modal -->
    <div 
      v-if="showAddModal" 
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
      @click.self="closeModal"
    >
      <div class="bg-white rounded-xl shadow-xl max-w-md w-full max-h-[90vh] overflow-y-auto">
        <!-- Modal Header -->
        <div class="flex items-center justify-between p-6 border-b border-gray-200">
          <h2 class="text-xl font-bold text-gray-900">เพิ่มหนังสือใหม่</h2>
          <button 
            @click="closeModal"
            class="text-gray-400 hover:text-gray-600 transition-colors duration-200"
          >
            <i class="pi pi-times text-xl"></i>
          </button>
        </div>

        <!-- Modal Body -->
        <div class="p-6">
          <form @submit.prevent="addBook" class="space-y-6">
            <!-- Title -->
            <div>
              <label for="bookTitle" class="block text-sm font-medium text-gray-700 mb-2">
                ชื่อหนังสือ <span class="text-red-500">*</span>
              </label>
              <input
                id="bookTitle"
                v-model="newBook.tille"
                type="text"
                required
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="กรอกชื่อหนังสือ"
              />
            </div>

            <!-- Author -->
            <div>
              <label for="bookAuthor" class="block text-sm font-medium text-gray-700 mb-2">
                ผู้แต่ง <span class="text-red-500">*</span>
              </label>
              <input
                id="bookAuthor"
                v-model="newBook.author"
                type="text"
                required
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="กรอกชื่อผู้แต่ง"
              />
            </div>

            <!-- Published Year -->
            <div>
              <label for="bookYear" class="block text-sm font-medium text-gray-700 mb-2">
                ปีที่ออกจำหน่าย
              </label>
              <input
                id="bookYear"
                v-model.number="newBook.published_year"
                type="number"
                min="1000"
                max="2100"
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="2024"
              />
            </div>

            <!-- Genre -->
            <div>
              <label for="bookGenre" class="block text-sm font-medium text-gray-700 mb-2">
                หมวดหมู่ <span class="text-red-500">*</span>
              </label>
              <input
                id="bookGenre"
                v-model="newBook.genre"
                type="text"
                required
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="กรอกหมวดหมู่หนังสือ"
              />
            </div>

            <!-- Submit Button -->
            <div class="flex space-x-3 pt-4">
              <button
                type="button"
                @click="closeModal"
                class="flex-1 px-4 py-3 border border-gray-300 text-gray-700 rounded-lg hover:bg-gray-50 font-medium transition-colors duration-200"
                :disabled="isSubmitting"
              >
                ยกเลิก
              </button>
              <button
                type="submit"
                class="flex-1 bg-blue-600 hover:bg-blue-700 disabled:bg-blue-400 text-white px-4 py-3 rounded-lg font-medium transition-colors duration-200 flex items-center justify-center space-x-2"
                :disabled="isSubmitting"
              >
                <div v-if="isSubmitting" class="animate-spin rounded-full h-4 w-4 border-2 border-white border-t-transparent"></div>
                <i v-else class="pi pi-check"></i>
                <span>{{ isSubmitting ? 'กำลังบันทึก...' : 'บันทึก' }}</span>
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Edit Book Modal -->
    <div 
      v-if="showEditModal" 
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
      @click.self="closeModal"
    >
      <div class="bg-white rounded-xl shadow-xl max-w-md w-full max-h-[90vh] overflow-y-auto">
        <!-- Modal Header -->
        <div class="flex items-center justify-between p-6 border-b border-gray-200">
          <h2 class="text-xl font-bold text-gray-900">แก้ไขหนังสือ</h2>
          <button 
            @click="closeModal"
            class="text-gray-400 hover:text-gray-600 transition-colors duration-200"
          >
            <i class="pi pi-times text-xl"></i>
          </button>
        </div>

        <!-- Modal Body -->
        <div class="p-6">
          <form @submit.prevent="updateBook" class="space-y-6">
            <!-- Title -->
            <div>
              <label for="editBookTitle" class="block text-sm font-medium text-gray-700 mb-2">
                ชื่อหนังสือ <span class="text-red-500">*</span>
              </label>
              <input
                id="editBookTitle"
                v-model="editBook.tille"
                type="text"
                required
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="กรอกชื่อหนังสือ"
              />
            </div>

            <!-- Author -->
            <div>
              <label for="editBookAuthor" class="block text-sm font-medium text-gray-700 mb-2">
                ผู้แต่ง <span class="text-red-500">*</span>
              </label>
              <input
                id="editBookAuthor"
                v-model="editBook.author"
                type="text"
                required
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="กรอกชื่อผู้แต่ง"
              />
            </div>

            <!-- Published Year -->
            <div>
              <label for="editBookYear" class="block text-sm font-medium text-gray-700 mb-2">
                ปีที่ออกจำหน่าย
              </label>
              <input
                id="editBookYear"
                v-model.number="editBook.published_year"
                type="number"
                min="1000"
                max="2100"
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="2024"
              />
            </div>

            <!-- Genre -->
            <div>
              <label for="editBookGenre" class="block text-sm font-medium text-gray-700 mb-2">
                หมวดหมู่ <span class="text-red-500">*</span>
              </label>
              <input
                id="editBookGenre"
                v-model="editBook.genre"
                type="text"
                required
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-colors duration-200"
                placeholder="กรอกหมวดหมู่หนังสือ"
              />
            </div>

            <!-- Submit Button -->
            <div class="flex space-x-3 pt-4">
              <button
                type="button"
                @click="closeModal"
                class="flex-1 px-4 py-3 border border-gray-300 text-gray-700 rounded-lg hover:bg-gray-50 font-medium transition-colors duration-200"
                :disabled="isSubmitting"
              >
                ยกเลิก
              </button>
              <button
                type="submit"
                class="flex-1 bg-green-600 hover:bg-green-700 disabled:bg-green-400 text-white px-4 py-3 rounded-lg font-medium transition-colors duration-200 flex items-center justify-center space-x-2"
                :disabled="isSubmitting"
              >
                <div v-if="isSubmitting" class="animate-spin rounded-full h-4 w-4 border-2 border-white border-t-transparent"></div>
                <i v-else class="pi pi-save"></i>
                <span>{{ isSubmitting ? 'กำลังอัพเดท...' : 'อัพเดท' }}</span>
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div 
      v-if="showDeleteModal" 
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
      @click.self="closeModal"
    >
      <div class="bg-white rounded-xl shadow-xl max-w-md w-full">
        <!-- Modal Header -->
        <div class="flex items-center justify-between p-6 border-b border-gray-200">
          <h2 class="text-xl font-bold text-gray-900 text-red-600">ยืนยันการลบ</h2>
          <button 
            @click="closeModal"
            class="text-gray-400 hover:text-gray-600 transition-colors duration-200"
          >
            <i class="pi pi-times text-xl"></i>
          </button>
        </div>

        <!-- Modal Body -->
        <div class="p-6">
          <div class="text-center">
            <div class="p-4 bg-red-100 rounded-full w-fit mx-auto mb-4">
              <i class="pi pi-exclamation-triangle text-red-600 text-3xl"></i>
            </div>
            <h3 class="text-lg font-medium text-gray-900 mb-2">
              คุณแน่ใจหรือไม่?
            </h3>
            <p class="text-gray-600 mb-2">
              คุณต้องการลบหนังสือ
            </p>
            <p class="font-medium text-gray-900 mb-6">
              "{{ deleteBook.tille }}"
            </p>
            <p class="text-red-600 text-sm">
              การดำเนินการนี้ไม่สามารถย้อนกลับได้
            </p>
          </div>

          <!-- Action Buttons -->
          <div class="flex space-x-3 pt-6">
            <button
              type="button"
              @click="closeModal"
              class="flex-1 px-4 py-3 border border-gray-300 text-gray-700 rounded-lg hover:bg-gray-50 font-medium transition-colors duration-200"
              :disabled="isSubmitting"
            >
              ยกเลิก
            </button>
            <button
              type="button"
              @click="confirmDelete"
              class="flex-1 bg-red-600 hover:bg-red-700 disabled:bg-red-400 text-white px-4 py-3 rounded-lg font-medium transition-colors duration-200 flex items-center justify-center space-x-2"
              :disabled="isSubmitting"
            >
              <div v-if="isSubmitting" class="animate-spin rounded-full h-4 w-4 border-2 border-white border-t-transparent"></div>
              <i v-else class="pi pi-trash"></i>
              <span>{{ isSubmitting ? 'กำลังลบ...' : 'ลบ' }}</span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url("https://unpkg.com/primeicons/primeicons.css");
@import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap');

body {
  font-family: 'Sarabun', sans-serif;
}

.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>