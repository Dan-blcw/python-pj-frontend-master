<script setup>
import { ref } from "vue";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";
import ModalPayment from "~/components/common/ModalPayment.vue";
import CartItem from "~/components/common/CartItem.vue";

const { $apiUrl } = useNuxtApp();

const auth = useAuthStore();

const tableData = ref([]);
const paymentInfo = ref(0);
const refModalPayment = ref();

const headerRowStyle = () => {
  return "header-cell-dio-style";
};

const calculateTotalCost = () => {
  paymentInfo.value = tableData.value.reduce(
    (total, item) => total + item.priceCart,
    0
  );
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
        // paymentInfo.value += item.product.price * item.quantity;
        const priceCart = item.product.price * item.quantity;
        return {
          ...item.product,
          stt: index + 1,
          id: item.id,
          idProduct: item.product.id,
          colors: item.colors,
          size: item.size,
          quantityCart: item.quantity,
          priceCart: priceCart,
        };
      });
      calculateTotalCost();
      // console.table(tableData.value)
    })
    .catch((error) => {
      console.log("error", error.response);
    });
};
const handleChange = (value, item, oldValue) => {
  if (!value) {
    getCart();
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
      item.quantityCart = value;
      item.priceCart = value * item.price;
      calculateTotalCost();
      getCart();
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
      calculateTotalCost();
      getCart();
      ElMessage.success("Xoá sản phấm thảnh công");
    })
    .catch((error) => {
      console.log("error", error.response);
      ElMessage.error("Xoá sản phấm thất bại");
    });
  getCart();
};

const openModal = () => {
  refModalPayment.value.openModal();
};

getCart();
</script>

<template>
  <div class="max-w-[1300px] mx-auto py-12">
    <h1
      class="text-[36px] font-semibold leading-tight pb-10 border-b-[1px] border-l-neutral-300 border-solid"
    >
      Giỏ hàng
    </h1>
    <div class="w-full flex flex-row justify-between">
      <div class="w-3/5 bg-white mt-5">
        <!-- StarT: Cart Item List -->
        <div class="w-full pr-10">
          <!-- Cart Item -->
          <CartItem
            v-for="item in tableData"
            :key="item.id"
            :item="item"
            :on-quantity-change="handleChange"
            :on-remove-item="deleteItemCart"
          />
          <!-- Cart Item -->
        </div>
        <!-- End: Cart Item List -->
      </div>
      <div class="w-[1px] bg-gray-300 mt-5"></div>
      <div class="w-[400px] mt-5">
        <div class="w-full pl-10">
          <h2 class="text-2xl font-semibold mb-5">Tổng hóa đơn</h2>
          <ul>
            <li
              class="flex justify-between py-4 border-b-[1px] border-solid border-gray-300"
            >
              <h3>Chi phí đơn hàng</h3>
              <span class="text-lg font-semibold text-[#2c2c2c]">{{
                paymentInfo.toLocaleString("vi", {
                  style: "currency",
                  currency: "VND",
                })
              }}</span>
            </li>
            <li
              class="flex justify-between py-4 border-b-[1px] border-solid border-gray-300"
            >
              <h3>Tiền vận chuyển</h3>
              <span class="text-lg font-semibold text-[#2c2c2c]"
                >30.000VNĐ</span
              >
            </li>
            <li
              class="flex justify-between py-4 border-b-[1px] border-solid border-gray-300"
            >
              <h3>Tiền thuế</h3>
              <span class="text-lg font-semibold text-[#2c2c2c]"
                >50.000VNĐ</span
              >
            </li>
            <li
              class="flex justify-between py-4 text-xl font-semibold text-[#2c2c2c]"
            >
              <h3>Tổng chi phí</h3>
              <span>1.080.000VNĐ</span>
            </li>
          </ul>
          <button
            class="w-full h-12 mt-5 font-medium bg-[#2c2c2c] text-white border-2 border-solid border-[#2c2c2c] text-lg rounded-full hover:text-[#2c2c2c] hover:bg-white transition-colors"
            @click="openModal"
          >
            Thanh toán
          </button>
        </div>
      </div>
    </div>
    <ModalPayment ref="refModalPayment" :order-details="tableData" />

    <!-- <div class="w-[600px] bg-slate-200 py-2 px-4 rounded-lg">
      <h2 class="text-xl p-2 text-center font-bold text-[#2c2c2c] mb-5">
        Thông tin thanh toán
      </h2>
      <div class="flex flex-row flex-wrap justify-between items-center">
        <div class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="name"
            >Họ và tên</label
          >
          <div>
            <input
              class="mt-1 px-4 py-2 rounded-lg border-2 border-solid border-gray-200 w-full outline-blue-500 transition-all"
              type="text"
              id="name"
              name="name"
              placeholder="Họ và tên..."
            />
          </div>
        </div>
        <div class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="email"
            >Email</label
          >
          <div>
            <input
              class="mt-1 px-4 py-2 rounded-lg border-2 border-solid border-gray-200 w-full outline-blue-500 transition-all"
              type="email"
              id="email"
              name="email"
              placeholder="Email..."
            />
          </div>
        </div>
      </div>

      <div class="flex flex-row flex-wrap justify-between items-center">
        <div class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="phoneNumber"
            >Số điện thoại</label
          >
          <div>
            <input
              class="mt-1 px-4 py-2 rounded-lg border-2 border-solid border-gray-200 w-full outline-blue-500 transition-all"
              type="text"
              id="phoneNumber"
              name="phoneNumber"
              placeholder="Số điện thoại..."
            />
          </div>
        </div>
        <div class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="address"
            >Địa chỉ</label
          >
          <div>
            <input
              class="mt-1 px-4 py-2 rounded-lg border-2 border-solid border-gray-200 w-full outline-blue-500 transition-all"
              type="text"
              id="address"
              name="address"
              placeholder="Địa chỉ..."
            />
          </div>
        </div>
      </div>
      <div class="mb-4 p-2">
        <h3 class="text-[#2c2c2c] text-base font-medium">
          Phương thức thanh toán
        </h3>
        <div class="mt-1 flex items-center gap-2">
          <input
            type="radio"
            id="paymentMethod_direct"
            name="paymentMethod"
            checked
            value="0"
          />
          <label class="cursor-pointer" for="paymentMethod_direct"
            >Thanh toán khi nhận hàng</label
          >
        </div>
        <div class="mt-1 flex items-center gap-2">
          <input
            type="radio"
            id="paymentMethod_banking"
            name="paymentMethod"
            value="1"
            disabled
          />
          <label class="cursor-pointer" for="paymentMethod_banking"
            >Chuyển khoản ngân hàng</label
          >
        </div>
        <div class="my-4">
          <label class="text-[#2c2c2c] text-base font-medium" for="message"
            >Tin nhắn
          </label>
          <div>
            <textarea
              name="message"
              id="message"
              placeholder="Mô tả..."
              rows="5"
              class="w-full mt-1 px-4 py-2 rounded-lg border-2 border-solid border-gray-200 outline-blue-500 transition-all"
            ></textarea>
          </div>
        </div>
        <p
          class="mb-4 flex text-[#2c2c2c] text-base font-medium items-center gap-2"
        >
          Tổng hóa đơn: <span class="text-lg">500.000.000VNĐ</span>
        </p>
        <button
          class="w-full rounded-full bg-slate-500 text-white h-10 text-xl border-2 border-solid border-slate-500 hover:text-slate-500 hover:bg-white transition-all"
        >
          Xác nhận
        </button>
      </div>
    </div> -->
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
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  margin: 0;
}
</style>
