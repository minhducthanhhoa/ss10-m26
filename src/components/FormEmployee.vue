<template>
  <!-- Form thêm mới nhân viên -->
  <div class="overlay">
    <form class="form" @submit.prevent="handleSubmit">
      <div class="d-flex justify-content-between align-items-center">
        <h4>Chỉnh sửa nhân viên</h4>
        <i @click="handleClose" class="fa-solid fa-xmark"></i>
      </div>
      <div>
        <label class="form-label" for="userName">Họ và tên</label>
        <input
          v-model="employee.name"
          id="userName"
          type="text"
          class="form-control"
        />
        <div v-if="nameError" class="form-text error">
          {{ nameError }}
        </div>
      </div>
      <div>
        <label class="form-label" for="dateOfBirth">Ngày sinh</label>
        <input
          v-model="employee.dateOfbBirth"
          id="dateOfBirth"
          type="date"
          class="form-control"
        />
      </div>
      <div v-if="dateOfBirthError" class="form-text error">
        {{ dateOfBirthError }}
      </div>
      <div>
        <label class="form-label" for="email">Email</label>
        <input
          v-model="employee.email"
          id="email"
          type="text"
          class="form-control"
        />
      </div>
      <div v-if="emailError" class="form-text error">
        {{ emailError }}
      </div>
      <div>
        <label class="form-label" for="address">Địa chỉ</label>
        <textarea
          v-model="employee.address"
          class="form-control"
          id="address"
          rows="3"
        ></textarea>
      </div>
      <div>
        <button class="w-100 btn btn-primary">Thêm mới</button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { reactive, ref } from "vue";
import { validateEmail } from "@/utils/validateData";

const { listEmployee } = defineProps(["listEmployee"]);

// Bước 1: Tạo emit
const emit = defineEmits("onClose");

// Hàm bắn emit
const handleClose = () => {
  emit("onClose");
};

const employee = reactive({
  id: Math.floor(Math.random() * 100000000),
  name: "",
  dateOfbBirth: "",
  email: "",
  address: "",
  status: true,
});

const nameError = ref("");
const emailError = ref("");
const dateOfBirthError = ref("");

const currentTime = () => {
  const today = new Date();

  const day = today.getDate().toString().padStart(2, 0);
  const month = (today.getMonth() + 1).toString().padStart(2, 0);
  const year = today.getFullYear();

  return `${year}-${month}-${day}`;
};

const handleSubmit = () => {
  console.log(employee);

  if (!employee.name) {
    nameError.value = "Tên không được để trống";
  } else {
    nameError.value = "";
  }

  if (employee.dateOfbBirth > currentTime()) {
    dateOfBirthError.value = "Ngày sinh không được lớn hơn ngày hiện tại";
  } else {
    dateOfBirthError.value = "";
  }

  if (!employee.email) {
    emailError.value = "Email không được để trống";
  } else {
    if (!validateEmail(employee.email)) {
      emailError.value = "Email không đúng định dạng";
    } else {
      emailError.value = "";
    }
  }

  if (
    employee.name &&
    employee.email &&
    validateEmail(employee.email) &&
    employee.dateOfbBirth <= currentTime()
  ) {
    // Push đối tượng employee lấy từ form vào mảng
    listEmployee.push(employee);

    // Lưu dữ liệu lên local
    localStorage.setItem("employees", JSON.stringify(listEmployee));

    // Đóng form
    handleClose();
  }
};
</script>

<style></style>