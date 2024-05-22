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
  <div class="max-w-[1300px] mx-auto py-12">
    <h1
      class="text-[36px] font-semibold leading-tight pb-10 border-b-2 border-l-neutral-300 border-solid"
    >
      Giỏ hàng
    </h1>
    <div class="w-full h-[1000px]">
      <div
        class="w-[60%] bg-white border-b-2 border-solid border-gray-300 mt-5"
      >
        <div class="w-ful p-5 h-48">
          <div class="flex-1 flex flex-row gap-5 w-full">
            <img
              class="w-32 h-36 rounded-lg border-2 border-solid border-gray-500"
              src="https://smakerclothing.com/thumb/320x300/1/upload/sanpham/dsc04412-1624.jpg"
              alt="product thumb"
            />
            <div class="flex-1 flex flex-col justify-between">
              <div class="flex flex-row flex-wrap justify-between">
                <div class="flex-1 flex flex-row flex-wrap justify-between">
                  <div>
                    <h3 class="text-base font-bold text-[#2c2c2c]">
                      LOGO CAMO PUFFER JACKET V2
                    </h3>
                    <div class="flex gap-5">
                      <span
                        class="pr-5 border-r-[1px] border-solid border-gray-200"
                        >Size: XL</span
                      >
                      <span>Color: Green</span>
                    </div>
                  </div>
                  <div
                    class="text-xl flex flex-row items-center h-10 border-2 border-solid border-gray-400"
                  >
                    <button class="w-10 h-8">-</button>
                    <span
                      class="w-10 h-full flex items-center justify-center border-x-2 border-solid border-gray-400"
                      >1</span
                    >
                    <button class="w-10 h-8">+</button>
                  </div>
                  <div><span class="text-xl">1.000.000VNĐ</span></div>
                </div>
              </div>
              <div class="flex justify-between items-center text-[#2c2c2c]">
                <span
                  class="text-sm px-4 py-1 border-[1px] border-solid border-gray-300 rounded-[99px]"
                  >Còn hàng</span
                >
                <span
                  class="text-lg px-4 py-1 border-[1px] border-solid border-gray-300 rounded-[99px] cursor-pointer"
                  >Xóa khỏi giỏ hàng</span
                >
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- <div class="w-full">
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
  </div> -->
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
