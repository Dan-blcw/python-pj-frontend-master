<script setup>
import { ref } from "vue";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";
import ModalPayment from "~/components/common/ModalPayment.vue";

// import { ref } from "vue";

const props = defineProps({
  item: {
    type: Object,
    required: true,
  },
  onQuantityChange: {
    type: Function,
    required: true,
  },
  onRemoveItem: {
    type: Function,
    required: true,
  },
});

const decreaseQuantity = () => {
  if (props.item.quantityCart > 1) {
    props.onQuantityChange(props.item.quantityCart - 1, props.item);
  }
};

const increaseQuantity = () => {
  props.onQuantityChange(props.item.quantityCart + 1, props.item);
};

const onQuantityChange = (event) => {
  const newValue = parseInt(event.target.value, 10);
  if (newValue && newValue > 0) {
    props.onQuantityChange(newValue, props.item);
  }
};

const removeItem = () => {
  props.onRemoveItem(props.item);
};
</script>

<template>
  <div
    class="flex flex-row gap-5 w-full pb-5 my-5 border-b-[1px] border-solid border-gray-300"
  >
    <img
      class="w-32 h-36 rounded-lg border-2 border-solid border-gray-500"
      :src="item.productThumb"
      alt="product thumb"
    />
    <div class="flex-1 flex flex-col justify-between">
      <div class="flex flex-row items-center justify-between">
        <div class="flex-1 flex flex-row flex-wrap justify-between">
          <div class="basis-2/5 max-w-2/5">
            <h3 class="text-base font-bold text-[#2c2c2c]">
              {{ item.productName }}
            </h3>
            <div class="flex gap-5">
              <span class="pr-5 border-r-[1px] border-solid border-gray-200">
                Size: {{ item.size }}
              </span>
              <span>Color: {{ item.colors }}</span>
            </div>
          </div>
          <div
            class="w-fit text-lg flex flex-row items-center h-10 border-2 border-solid border-gray-400"
          >
            <button
              class="w-10 h-full hover:bg-slate-500 hover:text-white transition-colors"
              @click="decreaseQuantity"
            >
              -
            </button>
            <input
              :value="item.quantityCart"
              type="number"
              min="1"
              class="w-10 h-full text-center border-x-2 border-solid border-gray-400 outline-none"
              @change="onQuantityChange"
            />
            <button
              class="w-10 h-full hover:bg-slate-500 hover:text-white transition-colors"
              @click="increaseQuantity"
            >
              +
            </button>
          </div>
          <div class="basis-1/5 max-w-1/5">
            <span class="text-xl">{{ item.priceCart }}VNĐ</span>
          </div>
        </div>
      </div>
      <div class="flex justify-between items-center text-[#2c2c2c]">
        <span
          class="text-sm px-4 py-1 border-[1px] border-solid border-gray-300 rounded-[99px]"
        >
          Còn hàng
        </span>
        <span
          class="text-base text-white font-semibold bg-red-500 px-4 py-1 border-2 border-solid border-red-500 rounded-[99px] cursor-pointer hover:bg-white hover:text-red-500 transition-colors"
          @click="removeItem"
        >
          Xóa khỏi giỏ hàng
        </span>
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
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  margin: 0;
}
</style>
