<script setup>
import { reactive, ref } from "vue";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";

const props = defineProps({
  orderDetails: {
    type: Array,
    default: [],
  },
});
const { $apiUrl } = useNuxtApp();
const auth = useAuthStore();
const dialogFormVisible = ref(false);
const formRef = ref();
const loading = ref(false);
const formLabelWidth = "200px";

const form = reactive({
  sku: "",
  name: auth.$state.user.username,
  first_name: auth.$state.user.first_name,
  last_name: auth.$state.user.last_name,
  email: auth.$state.user.email,
  phone: "",
  address: "",
  method: "1",
});

const rules = reactive({
  phone: [
    {
      required: true,
      message: "Vui lòng điền thông tin số điện thoại",
      trigger: "blur",
    },
  ],
  address: [
    {
      required: true,
      message: "Vui lòng điền thông tin địa chỉ",
      trigger: "blur",
    },
  ],
});
let result = 0;
const randomString = () => {
  const chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  for (let i = 0; i < 10; i++) {
    form.sku += chars.charAt(Math.floor(Math.random() * chars.length));
  }
};

const openModal = () => {
  dialogFormVisible.value = true;
};
const closeModal = () => {
  dialogFormVisible.value = false;
  formRef.value.resetFields();
};

const submit = async () => {
  await formRef.value.validate(async (valid, fields) => {
    if (valid) {
      loading.value = true;
      randomString();
      spaFetch()($apiUrl.ORDER, {
        method: "POST",
        body: {
          sku: form.sku,
          name: form.name,
          phone: form.phone,
          status: "Chờ xử lý",
          address: form.address,
          order_details: props.orderDetails.map((i) => {
            return {
              product: i.idProduct,
              quantity: i.quantityCart,
              colors: i.colors,
              size: i.size,
            };
          }),
        },
      })
        .then((res) => {
          deleteCart();
          ElMessage.success("Đặt hàng thành công");
        })
        .catch((error) => {
          console.log("error", error.response);
          loading.value = false;
          ElMessage.error("Đặt hàng thất bại");
        });
    }
  });
};

const deleteCart = () => {
  const id =
    auth.$state.user && auth.$state.user.cart ? auth.$state.user.cart : null;
  spaFetch()(`${$apiUrl.CART}${id}/`, {
    method: "DELETE",
  })
    .then((res) => {
      loading.value = false;
      window.location.reload();
    })
    .catch((error) => {
      console.log("error", error.response);
    });
};
watch(dialogFormVisible, (value) => {
  if (!value) {
    formRef.value.resetFields();
  }
});
defineExpose({
  openModal,
});
</script>

<template>
  <!-- <el-dialog width="600" v-model="dialogFormVisible">
    <div class="">
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
              id="phone"
              name="phone"
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
    </div>
  </el-dialog> -->
  <el-dialog
    v-model="dialogFormVisible"
    title="Thông tin thanh toán"
    width="600"
  >
    <el-form :model="form" ref="formRef" :rules="rules">
      <div class="flex flex-row flex-wrap justify-between items-center">
        <el-form-item prop="name" class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="name"
            >Họ và tên</label
          >
          <el-input v-model="form.name" autocomplete="off" disabled />
        </el-form-item>
        <el-form-item prop="email" class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="name"
            >Email</label
          >
          <el-input v-model="form.email" autocomplete="off" disabled />
        </el-form-item>
      </div>

      <!-- <div class="flex flex-row flex-wrap justify-between items-center">
        <el-form-item prop="first_name" class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="name"
            >Firstname</label
          >
          <el-input v-model="form.first_name" autocomplete="off" disabled />
        </el-form-item>
        <el-form-item prop="last_name" class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="name"
            >Lastname</label
          >
          <el-input v-model="form.last_name" autocomplete="off" disabled />
        </el-form-item>
      </div> -->

      <div class="flex flex-row flex-wrap justify-between items-center">
        <el-form-item prop="phone" class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="name"
            >Số điện thoại</label
          >
          <el-input v-model="form.phone" autocomplete="off" />
        </el-form-item>
        <el-form-item prop="address" class="basis-1/2 max-w-1/2 p-2">
          <label class="text-[#2c2c2c] text-base font-medium" for="name"
            >Địa chỉ</label
          >
          <el-input v-model="form.address" autocomplete="off" />
        </el-form-item>
      </div>
      <h3 class="text-[#2c2c2c] text-base font-medium">
        Phương thức thanh toán
      </h3>
      <div class="mb-4 p-2 flex flex-row">
        <div class="flex items-center gap-2 ms-5">
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
        <div class="flex items-center gap-2 ms-5">
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
    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="closeModal">Huỷ</el-button>
        <el-button type="primary" @click="submit" :loading="loading" to="/">
          Xác nhận
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<style lang="scss"></style>
