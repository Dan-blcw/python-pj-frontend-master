<script setup>
import { reactive, ref, watch } from "vue";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";
import { ElMessage } from "element-plus";

const props = defineProps({
  data: {
    type: Object,
    default: null,
  },
  isUpdate: {
    type: Boolean,
    default: false,
  },
});

const emits = defineEmits(["on-success"]);

const { $apiUrl } = useNuxtApp();
const auth = useAuthStore();
const dialogFormVisible = ref(false);
const formRef = ref();
const loading = ref(false);
const title = ref("Thông tin Đơn Hàng");

const formLabelWidth = "240px";
const epcung = "Thanh toán khi nhận được hàng";
const form = reactive({
  id: null,
  name: "",
  phone: "",
  address: "",
  total_amount: 0,
  create_at: "",
  user_id: 0,
  sku: "",
  status: "",
  pttt: "Thanh toán khi nhận được hàng",
});

const headerRowStyle = () => {
  return "header-cell-style";
};
const rules = reactive({
  name: [
    {
      required: true,
      message: "Vui lòng điền thông tin",
      trigger: "blur",
    },
  ],
  img: [
    {
      required: true,
      message: "Vui lòng điền thông tin",
      trigger: "blur",
    },
  ],
  type: [
    {
      required: true,
      message: "Vui lòng điền thông tin",
      trigger: "blur",
    },
  ],
});

const openModal = () => {
  dialogFormVisible.value = true;
};
const closeModal = () => {
  dialogFormVisible.value = false;
  formRef.value.resetFields();
};

const submit = async () => {
  await updateProduct();
};

const updateProduct = async () => {
  await formRef.value.validate(async (valid, fields) => {
    if (valid) {
      loading.value = true;
      spaFetch()(`${$apiUrl.ORDER}${form.id}/`, {
        method: "PATCH",
        body: {
          status: form.status,
        },
      })
        .then((res) => {
          emits("on-success");
          loading.value = false;
          dialogFormVisible.value = false;
          ElMessage.success("Cập nhật thành công");
        })
        .catch((error) => {
          console.log("error", error.response);
          loading.value = false;
          ElMessage.error("Cập nhật thất bại");
        });
    }
  });
};

const setFormState = (data) => {
  form.id = data.id;
  form.name = data.name;
  form.phone = data.phone;
  form.address = data.address;
  form.create_at = data.create_at;
  form.total_amount = data.total_amount;
  form.user_id = data.user_id;
  form.sku = data.sku;
  form.status = data.status;
};

// watch(dialogFormVisible, (value) => {
//   if (!value) {
//     formRef.value.resetFields();
//     if (form.id) delete form.id;
//   }
// });

// watch(
//   () => props.isUpdate,
//   (value) => {
//     title.value = value ? "Cập nhật sản phẩm" : "Thêm sản phẩm";
//   },
//   { immediate: true }
// );

defineExpose({
  openModal,
  setFormState,
});
</script>
<template>
  <el-dialog
    v-model="dialogFormVisible"
    :title="title"
    width="1200"
    :header-cell-class-name="headerRowStyle"
  >
    <el-form :model="form" ref="formRef" :rules="rules">
      <el-form-item prop="sku" label="Mã Hóa đơn" :label-width="formLabelWidth">
        <el-input v-model="form.sku" autocomplete="off" disabled />
      </el-form-item>
      <el-form-item
        prop="name"
        label="Tên khách hàng"
        :label-width="formLabelWidth"
      >
        <el-input v-model="form.name" autocomplete="off" disabled />
      </el-form-item>
      <el-form-item
        prop="phone"
        label="Số điện thoại"
        :label-width="formLabelWidth"
      >
        <el-input v-model="form.phone" autocomplete="off" disabled />
      </el-form-item>
      <el-form-item
        prop="address"
        label="Địa chỉ"
        :label-width="formLabelWidth"
      >
        <el-input v-model="form.address" autocomplete="off" disabled />
      </el-form-item>
      <el-form-item
        prop="status"
        label="Trạng thái"
        :label-width="formLabelWidth"
      >
        <!-- <el-input v-model="form.status" autocomplete="off" /> -->
        <!-- <div>{{ form.status }}</div> -->
        <select v-model="form.status" class="ms-3">
          <option value="">Please select one</option>
          <option>Chờ lấy hàng</option>
          <option>Đang đi lấy</option>
          <option>Đang giao hàng</option>
          <option>Đã giao hàng</option>
          <option>Chờ xử lý</option>
        </select>
      </el-form-item>
      <el-form-item
        prop="total_amount"
        label="Tổng giá trị đơn hàng"
        :label-width="formLabelWidth"
      >
        <el-input v-model="form.total_amount" autocomplete="off" disabled />
      </el-form-item>
      <el-form-item
        prop="pttt"
        label="Phương Thức Thanh toán"
        :label-width="formLabelWidth"
      >
        <el-input v-model="form.pttt" autocomplete="off" disabled />
      </el-form-item>
    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="closeModal">Huỷ</el-button>
        <el-button type="primary" @click="submit" :loading="loading"
          >Xác nhận</el-button
        >
      </div>
    </template>
  </el-dialog>
</template>

<style lang="scss">
.header-cell-style {
  background-color: #68c2c9 !important;
  color: black !important;
}
.text-center {
  text-align: center;
}
</style>
