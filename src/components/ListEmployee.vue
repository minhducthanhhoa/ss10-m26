<template>
  <div>
    <div class="w-[80%] m-auto mt-4 h-[100vh]">
      <main class="main">
        <header class="d-flex justify-content-between mb-3">
          <h3>Nhân viên</h3>
          <button @click="handleShowForm" class="btn btn-primary">
            Thêm mới nhân viên
          </button>
        </header>
        <div class="d-flex align-items-center justify-content-end gap-2 mb-3">
          <input
            v-model="searchEmail"
            @input="handleSearch"
            style="width: 350px"
            type="text"
            class="form-control"
            placeholder="Tìm kiếm theo email"
          />
          <i class="fa-solid fa-arrows-rotate" title="Refresh" @click="handleRefresh"></i>
        </div>
        <!-- Danh sách nhân viên -->
        <table class="table table-bordered table-hover table-striped">
          <thead>
            <tr>
              <th>STT</th>
              <th>Họ và tên</th>
              <th>Ngày sinh</th>
              <th>Email</th>
              <th>Địa chỉ</th>
              <th>Trạng thái</th>
              <th colspan="2">Chức năng</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(employeeItem, index) in paginatedEmployees"
              :key="employeeItem.id"
            >
              <td>{{ index + 1 + (currentPage - 1) * recordsPerPage }}</td>
              <td>{{ employeeItem.name }}</td>
              <td>{{ employeeItem.dateOfbBirth }}</td>
              <td>{{ employeeItem.email }}</td>
              <td>{{ employeeItem.address }}</td>
              <td>
                <div
                  v-if="employeeItem.status"
                  style="display: flex; align-items: center; gap: 8px"
                >
                  <div class="status status-active"></div>
                  <span> Đang hoạt động</span>
                </div>

                <div v-else style="display: flex; align-items: center; gap: 8px">
                  <div class="status status-stop"></div>
                  <span> Ngừng hoạt động</span>
                </div>
              </td>
              <td>
                <span class="button button-block" @click="handleBlock(employeeItem.id)">Chặn</span>
              </td>
              <td>
                <span class="button button-edit" @click="handleEdit(employeeItem)">Sửa</span>
              </td>
              <td><span class="button button-delete" @click="handleDelete(employeeItem.id)">Xóa</span></td>
            </tr>
          </tbody>
        </table>
        <footer class="d-flex justify-content-end align-items-center gap-3">
          <select class="form-select" v-model="recordsPerPage" @change="handlePagination">
            <option selected>Hiển thị 10 bản ghi trên trang</option>
            <option>Hiển thị 20 bản ghi trên trang</option>
            <option>Hiển thị 50 bản ghi trên trang</option>
            <option>Hiển thị 100 bản ghi trên trang</option>
          </select>
          <ul class="pagination">
            <li class="page-item" :class="{ disabled: currentPage === 1 }">
              <a class="page-link" href="#" @click="handlePreviousPage">Previous</a>
            </li>
            <li class="page-item" v-for="page in totalPages" :key="page" :class="{ active: currentPage === page }">
              <a class="page-link" href="#" @click="goToPage(page)">{{ page }}</a>
            </li>
            <li class="page-item" :class="{ disabled: currentPage === totalPages }">
              <a class="page-link" href="#" @click="handleNextPage">Next</a>
            </li>
          </ul>
        </footer>
      </main>
    </div>

    <!-- Modal xác nhận chặn tài khoản -->
    <div class="overlay" v-if="showBlockModal">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark" @click="closeBlockModal"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn chặn tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="closeBlockModal">Hủy</button>
          <button class="btn btn-danger" @click="confirmBlock">Xác nhận</button>
        </div>
      </div>
    </div>

    <!-- Modal xác nhận xóa tài khoản -->
    <div class="overlay" v-if="showDeleteModal">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark" @click="closeDeleteModal"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn xóa tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="closeDeleteModal">Hủy</button>
          <button class="btn btn-danger" @click="confirmDelete">Xác nhận</button>
        </div>
      </div>
    </div>

    <!-- Form quản lý nhân viên -->
    <FormEmployee
      v-if="isShowForm"
      @onClose="handleCloseForm"
      :listEmployee="listEmployee"
    />
  </div>
</template>

<script setup>
import { reactive, ref, computed } from "vue";
import FormEmployee from "./FormEmployee.vue";

// Biến dữ liệu danh sách nhân viên
const listEmployee = reactive(
  JSON.parse(localStorage.getItem("employees")) || []
);

// Biến hiển thị form
const isShowForm = ref(false);

// Biến tìm kiếm
const searchEmail = ref("");

// Biến modal chặn
const showBlockModal = ref(false);
let employeeToBlock = null;

// Biến modal xóa
const showDeleteModal = ref(false);
let employeeToDelete = null;

// Biến phân trang
const currentPage = ref(1);
const recordsPerPage = ref(10);

// Hàm mở form thêm mới
const handleShowForm = () => {
  isShowForm.value = true;
};

// Hàm đóng form
const handleCloseForm = () => {
  isShowForm.value = false;
};

// Hàm tìm kiếm theo email
const handleSearch = () => {
  // Lọc danh sách nhân viên theo email
  filteredEmployees.value = listEmployee.filter((employee) =>
    employee.email.toLowerCase().includes(searchEmail.value.toLowerCase())
  );
  currentPage.value = 1; // Reset trang về 1 khi tìm kiếm
};

// Hàm refresh danh sách nhân viên
const handleRefresh = () => {
  // Tải lại dữ liệu từ localStorage
  const employees = JSON.parse(localStorage.getItem("employees")) || [];
  listEmployee.splice(0, listEmployee.length, ...employees);
};

// Hàm hiển thị modal chặn
const handleBlock = (id) => {
  employeeToBlock = id;
  showBlockModal.value = true;
};

// Hàm xác nhận chặn nhân viên
const confirmBlock = () => {
  const employee = listEmployee.find((emp) => emp.id === employeeToBlock);
  if (employee) {
    employee.status = false;
    localStorage.setItem("employees", JSON.stringify(listEmployee));
  }
  closeBlockModal();
};

// Hàm đóng modal chặn
const closeBlockModal = () => {
  showBlockModal.value = false;
  employeeToBlock = null;
};

// Hàm hiển thị modal xóa
const handleDelete = (id) => {
  employeeToDelete = id;
  showDeleteModal.value = true;
};

// Hàm xác nhận xóa nhân viên
const confirmDelete = () => {
  const index = listEmployee.findIndex((emp) => emp.id === employeeToDelete);
  if (index !== -1) {
    listEmployee.splice(index, 1);
    localStorage.setItem("employees", JSON.stringify(listEmployee));
  }
  closeDeleteModal();
};

// Hàm đóng modal xóa
const closeDeleteModal = () => {
  showDeleteModal.value = false;
  employeeToDelete = null;
};

// Hàm sửa thông tin nhân viên
const handleEdit = (employee) => {
  // Mở form với thông tin nhân viên để chỉnh sửa
  isShowForm.value = true;
};

// Phân trang
const totalPages = computed(() => Math.ceil(listEmployee.length / recordsPerPage.value));
const paginatedEmployees = computed(() => {
  const start = (currentPage.value - 1) * recordsPerPage.value;
  const end = start + recordsPerPage.value;
  return listEmployee.slice(start, end);
});

// Hàm thay đổi trang
const goToPage = (page) => {
  currentPage.value = page;
};

// Hàm chuyển trang trước
const handlePreviousPage = () => {
  if (currentPage.value > 1) currentPage.value--;
};

// Hàm chuyển trang sau
const handleNextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++;
};

// Hàm thay đổi số bản ghi hiển thị trên mỗi trang
const handlePagination = () => {
  currentPage.value = 1; // Reset trang về 1 khi thay đổi số bản ghi
};
</script>

<style>

</style>
