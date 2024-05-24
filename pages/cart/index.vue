<script setup>
import { ref } from "vue";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";
import ModalPayment from "~/components/common/ModalPayment.vue";
import CartItem from "~/components/common/CartItem.vue";

const { $apiUrl } = useNuxtApp();

const auth = useAuthStore();

const cartList = ref([]);
const paymentCost = ref(0);

const refModalPayment = ref();

const headerRowStyle = () => {
  return "header-cell-dio-style";
};

const calculateTotalCost = () => {
  paymentCost.value = cartList.value.reduce(
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
      cartList.value = res.products.map((item, index) => {
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
      // console.table(cartList.value)
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
      cartList.value = cartList.value.filter((i) => i.id !== item.id);
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
        <div class="w-full pr-10" v-if="cartList.length > 0">
          <CartItem
            v-for="item in cartList"
            :key="item.id"
            :item="item"
            :on-quantity-change="handleChange"
            @quantity-change="handleChange"
            :on-remove-item="deleteItemCart"
          />
        </div>
        <div class="pr-10" v-else>
          <img
            class="max-w-full"
            src="https://www.chandranscreation.com/images/empty-cart.jpg"
            alt="cart empty"
          />
        </div>
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
                paymentCost.toLocaleString("vi", {
                  style: "currency",
                  currency: "VND",
                })
              }}</span>
            </li>
            <li
              class="flex justify-between py-4 border-b-[1px] border-solid border-gray-300"
            >
              <h3>Tiền vận chuyển</h3>
              <span class="text-lg font-semibold text-[#2c2c2c]">{{
                (cartList.length > 0 ? 30000 : 0).toLocaleString("vi", {
                  style: "currency",
                  currency: "VND",
                })
              }}</span>
            </li>
            <li
              class="flex justify-between py-4 border-b-[1px] border-solid border-gray-300"
            >
              <h3>Tiền thuế</h3>
              <span class="text-lg font-semibold text-[#2c2c2c]">{{
                (cartList.length > 0
                  ? (paymentCost * 5) / 100
                  : 0
                ).toLocaleString("vi", {
                  style: "currency",
                  currency: "VND",
                })
              }}</span>
            </li>
            <li
              class="flex justify-between py-4 text-xl font-semibold text-[#2c2c2c]"
            >
              <h3>Tổng chi phí</h3>
              <span>{{
                (cartList.length > 0
                  ? (paymentCost * 5) / 100 + 30000 + paymentCost
                  : 0
                ).toLocaleString("vi", {
                  style: "currency",
                  currency: "VND",
                })
              }}</span>
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
    <ModalPayment ref="refModalPayment" :order-details="cartList" />
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
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  margin: 0;
}
</style>
