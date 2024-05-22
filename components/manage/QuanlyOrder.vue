<script setup>
import { ref } from "vue";
import spaFetch from "~/plugins/fetch.js";
import { Search } from "@element-plus/icons-vue";
import ChitietOrder from "./ChitietOrder.vue";
import Update_status from "./Update_status.vue";
const { $apiUrl } = useNuxtApp();

const tableData = ref([]);
const loading = ref(false);
const refModalDetailCart = ref();
const search = ref("");
const isUpdate = ref(false);
const refUpdateModal = ref();

const handleEdit = (item) => {
  isUpdate.value = true;
  refUpdateModal.value.openModal();
  refUpdateModal.value.setFormState(item);
};
const getOrder = () => {
  loading.value = true;
  spaFetch()($apiUrl.ORDER, {
    method: "GET",
    params: {
      ordering: "-created_at",
      search: search.value,
      query_all: true,
    },
  })
    .then((res) => {
      tableData.value = res.map((item, index) => {
        return {
          stt: index + 1,
          ...item,
        };
      });
      loading.value = false;
    })
    .catch((error) => {
      loading.value = false;
      console.log("error", error.response);
    });
};

const openModal = (item) => {
  refModalDetailCart.value.openModal();
  refModalDetailCart.value.setTableData(item);
};

const handleSuccess = () => {
  search.value = "";
  getOrder();
};
getOrder();
watch(search, () => {
  setTimeout(() => {
    getOrder();
  }, 200);
});
</script>

<template>
  <div class="h-full flex flex-col gap-x-2 p-2">
    <div class="uppercase font-semibold text-2xl">Danh sách Đơn Hàng</div>
    <div class="search-container">
      <el-input
        v-model="search"
        placeholder="Tìm kiếm Đơn Hàng"
        :prefix-icon="Search"
        class="search-input custom-input"
      />
    </div>
    <el-table
      v-loading="loading"
      border
      :data="tableData"
      height="500"
      style="width: 100%"
    >
      <el-table-column prop="stt" label="STT" width="80"></el-table-column>
      <el-table-column prop="sku" label="Mã hóa đơn" width="150" />
      <el-table-column prop="name" label="khách hàng" width="120" />
      <el-table-column prop="phone" label="Số điện thoại" width="150" />
      <el-table-column prop="address" label="Địa chỉ giao hàng" width="340" />
      <el-table-column prop="status" label="Trạng Thái" width="140" />
      <el-table-column
        prop="total_amount"
        label="Tổng giá trị đơn hàng"
        width="200"
      />
      <el-table-column label="Phương thức thanh toán" width="240">
        Thanh toán khi nhận hàng
      </el-table-column>
      <el-table-column fixed="right" label="Hành động" width="180">
        <template #default="scope">
          <el-button type="primary" size="small" @click="openModal(scope.row)">
            Chi tiết
          </el-button>
          <el-button
            link
            type="primary"
            size="small"
            @click.prevent="handleEdit(scope.row)"
          >
            Thay đổi
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <span class="text-right mt-4 text-lg">
      Tổng số đơn hàng: {{ tableData.length }}
    </span>
    <ChitietOrder ref="refModalDetailCart" />
    <Update_status ref="refUpdateModal" @on-success="handleSuccess" />
  </div>
</template>

<style scoped>
.search-container {
  display: flex;
  align-items: center;
  gap: 8px; /* Adjust the gap between the input and the button */
  padding: 8px 0; /* Adjust the padding as needed */
}
</style>
