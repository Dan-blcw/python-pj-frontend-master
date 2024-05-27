<script setup>
import { ref } from "vue";
import ChitietOrder from "~/components/manage/ChitietOrder.vue";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";

const { $apiUrl } = useNuxtApp();

const auth = useAuthStore();

const tableData = ref([]);
const refModalPayment = ref();
// const refModalDetailCart = ref()
const headerRowStyle = () => {
  return "header-cell-style";
};

const getOrder = () => {
  spaFetch()($apiUrl.ORDER, {
    method: "GET",
    params: {
      user:
        auth.$state.user && auth.$state.user.id ? auth.$state.user.id : null,
      pageSize: 20,
    },
  })
    .then((res) => {
      tableData.value = res.results.map((item, index) => {
        return {
          stt: index + 1,
          ...item,
        };
      });
      // console.table(tableData.value)
    })
    .catch((error) => {
      console.log("error", error.response);
    });
};

const deleteOrder = (item) => {
  spaFetch()(`${$apiUrl.ORDER}${item.id}/`, {
    method: "DELETE",
  })
    .then((res) => {
      tableData.value = tableData.value.filter((i) => i.id !== item.id);
      ElMessage.success("Huỷ đơn hàng thảnh công");
    })
    .catch((error) => {
      console.log("error", error.response);
      ElMessage.error("Huỷ đơn hàng thất bại");
    });
};

const openModal = (item) => {
  refModalPayment.value.openModal();
  refModalPayment.value.setTableData(item);
};

getOrder();
</script>

<template>
  <div class="w-full">
    <div>
      <div class="flex-container flex items-center justify-center">
        <div class="w-11/12 mt-5 ms-5 me-5">
          <el-table
            :data="tableData"
            height="620"
            style="width: 100%"
            :header-cell-class-name="headerRowStyle"
          >
            <el-table-column
              prop="stt"
              label="STT"
              width="80"
            ></el-table-column>
            <el-table-column prop="sku" label="Mã hóa đơn" width="140" />
            <el-table-column prop="name" label="Họ Tên" width="110" />
            <el-table-column prop="phone" label="Số điện thoại" width="140" />
            <el-table-column prop="address" label="Địa chỉ" width="320" />
            <el-table-column prop="status" label="Trạng Thái" width="140" />
            <el-table-column
              prop="total_amount"
              label="Tổng giá trị đơn hàng"
              width="160"
            />
            <el-table-column label="Phương thức thanh toán">
              Thanh toán khi nhận hàng
            </el-table-column>
            <!-- <el-table-column>
              <template #default="prop">
                <el-button
                  type="danger"
                  class="uppercase"
                  @click="deleteOrder(prop.row)"
                >
                  Huỷ đơn hàng
                </el-button>
              </template>
            </el-table-column> -->

            <el-table-column width="auto" label="Hoạt động">
              <template #default="prop">
                <el-button
                  color="#82B7E3"
                  type="primary"
                  size="medium"
                  class=""
                  @click="openModal(prop.row)"
                >
                  Chi tiết
                </el-button>

                <el-popconfirm
                  title="Xác nhận chắc chắn muốn xóa đơn đặt hàng những sản phẩm này?"
                  width="300"
                  popper-class="custom-popconfirm  text-[50px]"
                  @confirm="deleteOrder(prop.row)"
                >
                  <template #reference>
                    <el-button color="#C58101" type="danger" size="medium">
                      xoá
                    </el-button>
                  </template>
                </el-popconfirm>
              </template>
            </el-table-column>
          </el-table>
          <ChitietOrder ref="refModalPayment" />
        </div>

        <div class="w-3/12">
          <img
            class="image"
            src="https://i.pinimg.com/564x/2c/ca/1d/2cca1d3ac636b17bdf5bc1cc7c6c63f9.jpg"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.header-cell-style {
  background-color: #c58101 !important;
  color: black !important;
}
.text-center {
  text-align: center;
}
.custom-popconfirm {
  /* Adjust the width */
  width: 300px !important;
  /* Additional custom styles */
  .el-popconfirm__main {
    padding: 20px; /* Example padding adjustment */
  }
}
</style>
