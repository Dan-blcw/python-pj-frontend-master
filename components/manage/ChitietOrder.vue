<script setup>
import { ref } from "vue";
import { useAuthStore } from "~/stores/auth.js";

const dialogFormVisible = ref(false);
const tableData = ref([]);
const auth = useAuthStore();
const headerRowStyle = () => {
  return "header-cell-style";
};

const setTableData = (data) => {
  tableData.value = data.order_details.map((item, index) => {
    return {
      quantityOrderDetail: item.quantity,
      priceOrderDetail: item.product.price * item.quantity,
      stt: index + 1,
      ...item.product,
    };
  });
};
const openModal = () => {
  dialogFormVisible.value = true;
};
const closeModal = () => {
  dialogFormVisible.value = false;
};
defineExpose({
  openModal,
  setTableData,
});
</script>

<template>
  <el-dialog
    v-model="dialogFormVisible"
    title="Thông tin Chi tiết Đơn Hàng"
    width="1200"
    align-center="true"
  >
    <el-table
      :data="tableData"
      height="500"
      style="width: 100%"
      :header-cell-class-name="headerRowStyle"
      show-summary
      sum-text="Tổng"
    >
      <el-table-column prop="stt" label="STT" width="80"></el-table-column>
      <el-table-column prop="img" label="Hình Ảnh" width="220">
        <template #default="prop">
          <img :src="prop.row.img" alt="" />
        </template>
      </el-table-column>
      <el-table-column prop="name" label="Sản phẩm" width="360" />
      <el-table-column prop="colors" label="Màu sắc" width="90" />
      <el-table-column prop="size" label="Kích thước" width="110" />
      <el-table-column
        prop="quantityOrderDetail"
        label="Số lượng"
        width="110"
      />
      <el-table-column prop="priceOrderDetail" label="Đơn giá" />
    </el-table>
  </el-dialog>
</template>

<style lang="scss">
.centered-dialog .el-dialog__body {
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
