<script setup>
import { ref } from "vue";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";
import ModalPayment from "~/components/common/ModalPayment.vue";

const { $apiUrl } = useNuxtApp();

const auth = useAuthStore();

const tableData = ref([]);
const refModalPayment = ref();

const headerRowStyle = () => {
  return "header-cell-dio-style";
};
const getCart = () => {
  const id =
    auth.$state.user && auth.$state.user.cart ? auth.$state.user.cart : null;
  spaFetch()(`${$apiUrl.CART}${id}/`, {
    method: "GET",
  })
    .then((res) => {
      auth.setQuantityInCart(res.products.length);
      tableData.value = res.products.map((item, index) => {
        return {
          ...item.product,
          stt: index + 1,
          id: item.id,
          idProduct: item.product.id,
          colors: item.colors,
          size: item.size,
          quantityCart: item.quantity,
          priceCart: item.product.price * item.quantity,
        };
      });
      // console.table(tableData.value)
    })
    .catch((error) => {
      console.log("error", error.response);
    });
};
const handleChange = (value, item, oldValue) => {
  if (!value) {
    return (item.quantityCart = oldValue);
  }
  spaFetch()(`${$apiUrl.CART_ITEM}${item.id}/`, {
    method: "PATCH",
    body: {
      quantity: value,
      colors: item.colors,
      size: item.size,
    },
  })
    .then((res) => {
      item.priceCart = value * item.price;
    })
    .catch((error) => {
      item.quantityCart = oldValue;
      console.log("error", error.response);
      if (error.status === 400) {
        ElMessage.error({
          message: "Số lượng hàng trong kho không đủ",
        });
      } else {
        ElMessage.error("Thêm vào giỏ hàng thất bại");
      }
    });
};

const deleteItemCart = (item) => {
  spaFetch()(`${$apiUrl.CART_ITEM}${item.id}/`, {
    method: "DELETE",
  })
    .then((res) => {
      tableData.value = tableData.value.filter((i) => i.id !== item.id);
      auth.setQuantityInCart(auth.$state.quantityInCart - 1);
      ElMessage.success("Xoá sản phấm thảnh công");
    })
    .catch((error) => {
      console.log("error", error.response);
      ElMessage.error("Xoá sản phấm thất bại");
    });
};

const openModal = () => {
  refModalPayment.value.openModal();
};

getCart();
</script>

<template>
  <div class="w-full">
    <div>
      <div class="flex-container flex items-center justify-center">
        <div class="flex w-2/5">
          <img
            class="image"
            src="https://i.pinimg.com/564x/c3/ab/57/c3ab5768c875ff3d83222efd4a94a68e.jpg"
          />
        </div>

        <div class="w-full flex items-center justify-center">
          <div class="w-11/12 mt-0">
            <el-table
              :data="tableData"
              height="750"
              style="width: 100%"
              :header-cell-class-name="headerRowStyle"
              show-summary
              sum-text="Tổng"
            >
              <el-table-column
                prop="stt"
                label="STT"
                width="56"
              ></el-table-column>
              <el-table-column
                prop="img"
                label="Hình Ảnh"
                width="220"
                class="text-center"
              >
                <template #default="prop">
                  <img :src="prop.row.img" alt="" />
                </template>
              </el-table-column>
              <el-table-column
                prop="name"
                label="Sản phẩm"
                class="text-center"
              ></el-table-column>
              <el-table-column
                prop="colors"
                label="Màu sắc"
                width="120"
                class="text-center"
              >
              </el-table-column>
              <el-table-column
                prop="size"
                label="Kích thước"
                width="120"
                class="text-center"
              ></el-table-column>
              <el-table-column
                prop="quantityCart"
                label="Số lượng "
                class="text-center"
              >
                <template #default="prop">
                  <el-input-number
                    v-model="prop.row.quantityCart"
                    :min="1"
                    @change="
                      (value, oldValue) =>
                        handleChange(value, prop.row, oldValue)
                    "
                  />
                </template>
              </el-table-column>
              <el-table-column
                prop="priceCart"
                label="Đơn giá"
                class="text-center"
              ></el-table-column>
              <el-table-column width="100" label="Hoạt động">
                <template #default="prop">
                  <el-button
                    color="#61C2C9"
                    type="danger"
                    class="uppercase w-[60px] h-[32px] text-[12px]"
                    @click="deleteItemCart(prop.row)"
                  >
                    xoá
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="w-full flex justify-center mt-4 mb-12">
              <el-button
                color="#61C2C9"
                type="success"
                class="uppercase w-[240px] h-[40px] text-[18px]"
                @click="openModal"
              >
                Thanh Toán
              </el-button>
            </div>
          </div>
          <ModalPayment ref="refModalPayment" :order-details="tableData" />
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.header-cell-dio-style {
  background-color: #68c2c9 !important;
  color: black !important;
}
.text-center {
  text-align: center;
}
</style>
