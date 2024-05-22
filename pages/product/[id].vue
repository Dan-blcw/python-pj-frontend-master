<script setup>
import { Monitor, ShoppingCart } from "@element-plus/icons-vue";
import { ref } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import { Autoplay, Navigation } from "swiper/modules";
import { fakeProducts } from "~/constants/fakeData.js";
import spaFetch from "~/plugins/fetch.js";
import { useAuthStore } from "~/stores/auth.js";

const { $apiUrl } = useNuxtApp();
const route = useRoute();
const router = useRouter();
var selectedColor = "Brown";
var selectedSize = "S";
const id = route.params ? route.params.id : null;

const modules = ref([Autoplay, Navigation]);
const products = ref(fakeProducts);
const dataDetail = ref(null);
const loading = ref(false);
const loadingAdd = ref(false);
const auth = useAuthStore();

const handleClick = (item) => {
  router.push(item && item.id ? `/product/${item.id}` : "#");
};

const num = ref(1);
const textarea2 = ref("");
const activeName = ref("first");
const getProducts = () => {
  spaFetch(false)($apiUrl.PRODUCT, {
    method: "GET",
    params: {
      page: 3,
      pageSize: 10,
    },
  })
    .then((res) => {
      products.value = res.results.map((item) => {
        return {
          ...item,
          status: item.type,
        };
      });
    })
    .catch((error) => {
      console.log("error", error.response);
    });
};
const getDetail = () => {
  loading.value = true;
  spaFetch(false)(`${$apiUrl.PRODUCT}${id}/`, {
    method: "GET",
  })
    .then((res) => {
      dataDetail.value = res;
      loading.value = false;
    })
    .catch((error) => {
      console.log("error", error.response);
      loading.value = false;
    });
};

const zoomIn = (event) => {
  const pre = document.getElementById("preview");
  pre.style.visibility = "visible";
  if ($("#zoom1").is(":hover")) {
    pre.style.backgroundImage = `url('${
      dataDetail.value && dataDetail.value.img ? dataDetail.value.img : null
    }')`;
  }
  const posX = event.offsetX;
  const posY = event.offsetY;
  pre.style.backgroundPosition = -posX * 3.8 + "px " + -posY * 3.5 + "px";
};

const zoomOut = () => {
  const pre = document.getElementById("preview");
  pre.style.visibility = "hidden";
};

const handleAddProductToCart = () => {
  if (!auth.$state.accessToken || !auth.$state.refreshToken) {
    return router.push("/login");
  }
  if (loadingAdd.value) {
    return;
  }
  loadingAdd.value = true;
  spaFetch()($apiUrl.CART_ITEM, {
    method: "POST",
    body: {
      product: id,
      quantity: num.value,
      colors: selectedColor,
      size: selectedSize,
    },
  })
    .then((res) => {
      ElMessage.success("Thêm vào giỏ hàng thành công");
      loadingAdd.value = false;
      getUser();
    })
    .catch((error) => {
      console.log("error", error.response);
      if (error.status === 400) {
        ElMessage.error({
          message: "Số lượng hàng trong kho không đủ",
        });
      } else {
        ElMessage.error("Thêm vào giỏ hàng thất bại");
      }
      loadingAdd.value = false;
    });
};

const handleChange = (value, oldValue) => {
  if (!value) {
    return (num.value = oldValue);
  }
};
const getUser = () => {
  const idUser =
    auth.$state.user && auth.$state.user.id ? auth.$state.user.id : null;
  spaFetch()(`${$apiUrl.USER}${idUser}/`, {
    method: "GET",
  })
    .then((res) => {
      auth.setUser(res);
      getCart();
    })
    .catch((error) => {
      console.log("error", error.response);
    });
};

const getCart = () => {
  const id =
    auth.$state.user && auth.$state.user.cart ? auth.$state.user.cart : null;
  spaFetch()(`${$apiUrl.CART}${id}/`, {
    method: "GET",
  })
    .then((res) => {
      auth.setQuantityInCart(res.products.length);
    })
    .catch((error) => {
      console.log("error", error.response);
    });
};

getCart();
getDetail();
getProducts();
</script>

<template>
  <div class="flex flex-col max-w-[1050px] mx-auto">
    <div class="w-full flex flex-col items-center justify-center mx-auto">
      <div v-loading="loading" class="w-full flex mt-4 justify-center gap-10">
        <div class="flex w-1/2 basis-1/2">
          <div class="w-full bg-white">
            <div id="samples" class="relative h-full max-w-full">
              <img
                id="zoom1"
                class="h-full w-auto object-cover"
                :src="dataDetail && dataDetail.img ? dataDetail.img : ''"
                alt=""
                @mousemove="zoomIn"
                @mouseout="zoomOut"
              />
              <div id="preview"></div>
            </div>
          </div>
        </div>

        <div class="ml-9 flex flex-col gap-y-2 w-1/2 basis-1/2">
          <div class="text-[1.85rem] text-[#0E7069] font-semibold">
            {{ dataDetail && dataDetail.name ? dataDetail.name : "" }}
          </div>

          <div class="text-[1.00rem] text-black font-semibold italic">
            Sku : AK0064-02
            <!-- {{ dataDetail && dataDetail.name ? dataDetail.name : '' }} -->
          </div>
          <div class="text-[1.10rem] text-black font-semibold italic">
            THÔNG TIN SẢN PHẨM:
            <!-- {{ dataDetail && dataDetail.name ? dataDetail.name : '' }} -->
          </div>

          <div class="text-black italic text-[1.00rem]">
            {{
              dataDetail && dataDetail.description ? dataDetail.description : ""
            }}
          </div>

          <div class="flex gap-5 items-center mb-1">
            <div
              class="flex text-black text-lg items-baseline text-[1.00rem] font-semibold"
            >
              Giá tiền :
              <div class="font-semibold text-teal-700 text-lg ml-4">
                {{ dataDetail && dataDetail.price ? dataDetail.price : 0 }} VND
              </div>
            </div>

            <div
              class="flex text-black text-lg items-baseline text-[1.00rem] font-semibold"
            >
              Trạng thái :
              <div class="font-semibold text-amber-700 text-lg ml-4 uppercase">
                Còn Hàng
              </div>
            </div>
          </div>

          <div
            class="flex text-black text-lg items-baseline text-[1.00rem] font-semibold mb-1"
          >
            Bộ sưu tầm :
            <div
              class="font-semibold text-teal-700 text-lg ml-4 cursor-pointer"
            >
              {{ dataDetail && dataDetail.type ? dataDetail.type : "" }}
            </div>
          </div>

          <!-- <div class="flex text-black text-lg items-baseline flex-wrap gap-y-1 text-[1.00rem]">
                        Từ khoá :
                        <div class="font-semibold text-[1.00rem] text-white text-lg ml-4 bg-blue-600 px-2 rounded-lg hover:bg-blue-500">
                            {{ dataDetail && dataDetail.type ? dataDetail.type : '' }}
                        </div>
                        <div class="font-semibold text-white text-lg ml-4 bg-blue-600 px-2 rounded-lg hover:bg-blue-500">
                            {{ dataDetail && dataDetail.name ? dataDetail.name : '' }}
                        </div>
                    </div> -->

          <!-- Color Selection -->
          <div class="flex flex-col gap-y-1">
            <div class="text-black text-lg text-[1.05rem] font-semibold">
              Chọn màu sắc:
            </div>
            <div class="flex gap-x-4">
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedColor"
                  name="colorProduct"
                  value="Brown"
                />
                <span class="w-6 h-6 rounded-full bg-brown inline-block"></span>
                Brown
              </label>
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedColor"
                  name="colorProduct"
                  value="Black"
                />
                <span class="w-6 h-6 rounded-full bg-black inline-block"></span>
                Black
              </label>
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedColor"
                  name="colorProduct"
                  value="Gray"
                />
                <span
                  class="w-6 h-6 rounded-full bg-gray-300 inline-block"
                ></span>
                Gray
              </label>
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedColor"
                  name="colorProduct"
                  value="Blue"
                />
                <span
                  class="w-6 h-6 rounded-full bg-sky-700 inline-block"
                ></span>
                Blue
              </label>
            </div>
          </div>

          <!-- Size Selection -->
          <div class="flex flex-col gap-y-1 mt-4">
            <div class="text-black text-lg text-[1.05rem] font-semibold">
              Chọn kích thước:
            </div>
            <div class="flex gap-x-4">
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedSize"
                  name="sizeProduct"
                  value="S"
                  @change="handleSizeChange"
                />
                S
              </label>
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedSize"
                  name="sizeProduct"
                  value="M"
                  @change="handleSizeChange"
                />
                M
              </label>
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedSize"
                  name="sizeProduct"
                  value="L"
                  @change="handleSizeChange"
                />
                L
              </label>
              <label class="flex items-center gap-x-1 cursor-pointer">
                <input
                  type="radio"
                  v-model="selectedSize"
                  name="sizeProduct"
                  value="XL"
                  @change="handleSizeChange"
                />
                XL
              </label>
            </div>
          </div>
          <div></div>

          <div class="flex items-center gap-5">
            <div class="h-full">
              <el-input-number
                class="h-[45px]"
                v-model="num"
                size="large"
                :min="1"
                @change="handleChange"
              />
            </div>
            <div
              class="flex h-12 bg-gray-800 text-white border-solid border-gray-800 border-2 items-center translate-x-4 justify-center px-10 gap-x-4 hover:bg-white hover:text-gray-800 cursor-pointer rounded-md transition-colors"
              @click="handleAddProductToCart"
              v-loading="loadingAdd"
            >
              <el-icon size="24">
                <ShoppingCart />
              </el-icon>
              <p class="uppercase text-base">Thêm vào giỏ</p>
            </div>
          </div>
        </div>
      </div>

      <div class="w-full translate-x-[0%]">
        <div>
          <div class="flex-container flex items-start">
            <div class="table-container flex-1">
              <div
                class="text-black text-lg text-[1.05rem] font-semibold relative"
              >
                Hướng dẫn cách đo kích thước sản phẩm:
              </div>
              <table>
                <thead>
                  <tr>
                    <th>Size</th>
                    <th>Vai</th>
                    <th>Ngực</th>
                    <th>Tay Áo</th>
                    <th>Dài Áo</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td>S</td>
                    <td>42</td>
                    <td>104</td>
                    <td>57</td>
                    <td>64</td>
                  </tr>
                  <tr>
                    <td>M</td>
                    <td>44</td>
                    <td>108</td>
                    <td>58</td>
                    <td>66</td>
                  </tr>
                  <tr>
                    <td>L</td>
                    <td>46</td>
                    <td>112</td>
                    <td>59</td>
                    <td>68</td>
                  </tr>
                  <tr>
                    <td>XL</td>
                    <td>48</td>
                    <td>116</td>
                    <td>60</td>
                    <td>70</td>
                  </tr>
                </tbody>
              </table>
              <p>
                *Dữ liệu này có được bằng cách đo thủ công sản phẩm, các phép đo
                có thể bị thay đổi 1-2 CM.
              </p>
              <div
                class="text-black text-lg text-[1.05rem] font-semibold relative mt-0 mb-1 ms-5"
              >
                Cách đo kích thước sản phẩm ?
              </div>
              <p class="mb-1">
                1.Vai____________Đo Từ đỉnh vai bên này sang bên kia.
              </p>
              <p class="mb-1">
                2. Ngực_________Từ nách hạ xuống 2cm, đo thẳng từ nách bên này
                sang nách bên kia.
              </p>
              <p class="mb-1">
                3.Chiều dài_____ Đo thân trước từ sát mí chân cổ trước đến lai
                áo.
              </p>
              <p class="mb-1">
                4.Tay áo________Đo từ đỉnh vai đến cuối tay áo.
              </p>
            </div>
            <div class="flex w-1/3">
              <img
                class="image"
                src="https://file.hstatic.net/1000026602/file/icon-02_700ef0b03438414f812433a5b797f044.jpg"
              />
            </div>
          </div>
        </div>
      </div>

      <!--Huong dan do trang suc-->
      <div class="w-full translate-x-[%]">
        <div>
          <div class="flex-container flex items-start">
            <div class="flex w-5/12 relative">
              <img
                class="image"
                src="https://www.pnj.com.vn/blog/wp-content/uploads/2021/11/huong-dan-do-size-nhan-3-1024x768.jpg"
              />
            </div>
            <div class="table-container flex-1 ms-5">
              <div
                class="text-black text-lg text-[1.05rem] font-semibold reative"
              >
                Hướng dẫn cách đo kích thước trang sức:
              </div>
              <table>
                <thead>
                  <tr>
                    <th>Loại Trang Sức</th>
                    <th>Kích Thước</th>
                    <th>Ghi Chú</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td>Vòng Cổ</td>
                    <td>35-50 cm</td>
                    <td>
                      Chiều dài từ S(35-30) M(40-45) L(50-52) XL(52-53) tùy loại
                      vòng
                    </td>
                  </tr>
                  <tr>
                    <td>Nhẫn</td>
                    <td>Size 5-10</td>
                    <td>Đo theo size ngón tay S(5-6) M(7) L(8) XL(9-10)</td>
                  </tr>
                  <tr>
                    <td>Vòng Tay</td>
                    <td>15-22 cm</td>
                    <td>
                      Chiều dài từ S(15-16) M(17-19) L(19-20) XL(20-22) tùy loại
                      vòng
                    </td>
                  </tr>
                </tbody>
              </table>
              <p>
                *Dữ liệu này có được bằng cách đo thủ công trang sức, các phép
                đo có thể bị thay đổi 0.2 - 0.5 CM. Hãy liên hệ với Shop để nhận
                được tư vấn
              </p>
              <div
                class="text-black text-lg text-[1.05rem] font-semibold relative mt-0 mb-1"
              >
                Cách đo kích thước trang sức?
              </div>
              <p class="mb-1">
                1. Vòng Cổ_________Dùng thước dây đo chiều dài từ khóa đến hết
                vòng.
              </p>
              <p class="mb-1">
                2. Nhẫn____________Dùng thước dây hoặc sợi chỉ đo chu vi ngón
                tay.
              </p>
              <p class="mb-1">
                3. Vòng Tay_______Dùng thước dây đo chiều dài từ đầu này đến đầu
                kia của vòng.
              </p>
            </div>
          </div>
        </div>
      </div>
      <!--  -->
      <!-- Start Danh gia -->
      <div class="w-11/12 mt-4 translate-x-[-0%]">
        <el-tabs
          v-model="activeName"
          type="border-card"
          class="demo-tabs"
          stretch
        >
          <el-tab-pane label="Đánh giá" name="first">
            <div class="comments-header">
              <p>2 bình luận</p>
              <p>Sắp xếp theo</p>
            </div>
            <div class="comment">
              <p class="comment-author">Michael</p>
              <p class="comment-text">
                Dry -EX does it again. During this hot summer, shirt is very
                comfortable to wear.
              </p>
              <p class="comment-date">17/05/2024</p>
            </div>
            <div class="comment">
              <p class="comment-author">Dan</p>
              <p class="comment-text">
                The design is great and has good material, the fit is
                comfortable and true to size, would definitely reccomend to
                others
              </p>
              <p class="comment-date">26/05/2024</p>
            </div>

            <el-input
              v-model="textarea2"
              :autosize="{ minRows: 2, maxRows: 4 }"
              type="textarea"
              placeholder="Nhập bình luận ..."
            />
            <div class="text-right">
              <el-button type="primary" plain>Đăng</el-button>
            </div>
          </el-tab-pane>

          <el-tab-pane label="Tin tức Mẹo" name="fourth">
            <div class="news-section">
              <h2 class="news-title">Các bài viết mới</h2>
              <div class="bank-info">
                <h3 class="news-item-title">
                  5 Mẹo giúp tăng tuổi thọ của thiết bị điện tử
                </h3>
                <p class="news-item-description">
                  Hãy áp dụng những mẹo nhỏ sau để thiết bị điện tử của bạn có
                  tuổi thọ cao hơn.
                </p>
              </div>
              <div class="bank-info">
                <h3 class="news-item-title">
                  Cách chọn máy tính xách tay phù hợp với nhu cầu sử dụng của
                  bạn
                </h3>
                <p class="news-item-description">
                  Đây là những gợi ý giúp bạn chọn được chiếc laptop phù hợp với
                  công việc và sở thích cá nhân.
                </p>
              </div>
              <div class="bank-info">
                <h3 class="news-item-title">
                  Những xu hướng công nghệ nổi bật năm 2024
                </h3>
                <p class="news-item-description">
                  Cùng khám phá những xu hướng công nghệ mới nhất và đang được
                  chú ý trong năm 2024.
                </p>
              </div>
              <!-- Add more news items as needed -->
            </div>
          </el-tab-pane>

          <el-tab-pane label="Thanh toán" name="payment">
            <div class="payment-section">
              <h2 class="payment-title">Thông tin thanh toán</h2>
              <div class="bank-info">
                <h3 class="branch-header">CHI NHÁNH HỒ CHÍ MINH</h3>
                <p class="bank-name">Ngân hàng: VIETCOMBANK</p>
                <p class="account-number">
                  <span class="label">Số tài khoản:</span> 83860101002
                </p>
                <p class="account-holder">
                  <span class="label">Chủ tài khoản:</span> Dan
                </p>
              </div>
              <div class="bank-info">
                <h3 class="branch-header">CHI NHÁNH HÀ NỘI</h3>
                <p class="bank-name">
                  Ngân hàng: VIETCOMBANK (Chi nhánh: Tây Hà Nội)
                </p>
                <p class="account-number">
                  <span class="label">Số tài khoản:</span> 83860101002
                </p>
                <p class="account-holder">
                  <span class="label">Chủ tài khoản:</span> Dan
                </p>
                <p class="note">
                  * Lưu ý: Vui lòng chuyển khoản theo định dạng: "Số điện thoại"
                  - Sau khi chuyển khoản, vui lòng liên hệ chúng tôi qua Zalo
                  hoặc trang fan page để xác nhận.
                </p>
              </div>
            </div>
          </el-tab-pane>
        </el-tabs>
      </div>
      <!-- End: danh gia -->

      <!-- Goi y san pham -->
      <div class="w-full mt-6 mb-6">
        <div
          class="w-2/5 bg-cyan-700 translate-x-[75%] flex h-[45px] justify-center items-center text-white font-semibold text-lg rounded-xl"
        >
          SẢN PHẨM KHÁC
        </div>
        <swiper
          :loop="true"
          :slides-per-view="4"
          :spaceBetween="20"
          :autoplay="{
            delay: 1500,
            disableOnInteraction: true,
          }"
          :navigation="true"
          :modules="modules"
          class="w-full"
        >
          <swiper-slide
            v-for="(product, index) in products"
            :key="index"
            class="mt-4 hover:mt-1 card-new-product"
          >
            <div
              class="w-full bg-white border border-solid border-gray-300 rounded-xl overflow-hidden cursor-pointer"
              @click="handleClick(product)"
            >
              <div class="w-full relative">
                <img class="w-full h-full" :src="product.img" alt="" />
              </div>
              <p
                class="w-full h-[40px] mt-2 text-sm px-2 font-normal hover:text-blue-800 line-clamp-2"
              >
                {{ product.name }}
              </p>
              <span class="p-2 border-solid text-black font-semibold">
                {{
                  product.price.toLocaleString("vi", {
                    style: "currency",
                    currency: "VND",
                  })
                }}
              </span>
              <div
                class="py-2 hover text-center font-bold text-blue-600 uppercase flex justify-center items-center gap-1 hover:text-white hover:bg-black transition duration-200"
              >
                <el-icon size="20"> <ShoppingCart /> </el-icon>
                Thêm vào giỏ
              </div>
            </div>
          </swiper-slide>
        </swiper>
      </div>
      <!-- </CategoryProducts> -->
    </div>
  </div>
</template>

<style lang="scss">
// .demo-tabs > .el-tabs__content {
//   padding: 32px;
//   color: #6b778c;
//   font-size: 15px;
//   font-weight: 600;
// }
.flex-container {
  display: flex;
  align-items: center;
}
.table-container {
  flex: 1;
}
.image-container {
  margin-left: 20px; /* Optional: Adjust the spacing between the table and image */
}

// .demo-tabs .el-tabs__item {
//   font-size: 18px;
//   font-weight: 300;
// }

// .el-descriptions__title {
//   color: red;
// }
.demo-tabs {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
}
.demo-tabs .el-tab-pane {
  padding: 20px;
}
.demo-tabs .el-tabs__header {
  margin-bottom: 5px;
}
.comments-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  font-size: 16px;
  font-weight: 600;
}
.comment {
  background: #fff;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.comment-author {
  font-weight: bold;
  margin-bottom: 5px;
}
.comment-text {
  margin-bottom: 5px;
}
.comment-date {
  font-size: 12px;
  color: #888;
}
.el-input {
  width: 100%;
  margin-bottom: 20px;
}
.text-right {
  text-align: right;
}
.el-button {
  border-radius: 5px;
}
.el-descriptions__title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
}
.el-descriptions-item {
  margin-bottom: 10px;
}
.branch-header {
  color: #ff0000;
  font-weight: bold;
  font-size: 18px;
  margin-top: 20px;
}
.bank-info {
  font-weight: bold;
}
.note {
  color: #ff0000;
  font-size: 16px;
  margin-top: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
}
.demo-tabs {
  background: #f9f9f9;
  padding: 12px;
  border-radius: 10px;
}
.news-section {
  padding: 12px;
}
.news-title {
  font-size: 28px;
  font-family: cursive;
  font-weight: bold;
  margin-bottom: 20px;
}
.news-item {
  margin-bottom: 20px;
}
.news-item-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 5px;
}
.news-item-description {
  color: #666;
}

table,
th,
td {
  border: 1px solid black;
}

th,
td {
  padding: 10px;
  text-align: center;
}

th {
  background-color: #f2f2f2;
}

h2 {
  margin-top: 20px;
}

ol {
  margin-left: 20px;
}
/* Styles for the new payment tab */
.payment-section {
  background-color: #f9f9f9;
  padding: 12px;
  border-radius: 8px;
}
.payment-title {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  margin-bottom: 16px;
}
.branch-header {
  font-size: 20px;
  font-weight: bold;
  color: #2a97a2;
  margin-bottom: 10px;
}
.bank-info {
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 8px;
}
.bank-name {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
}
.account-number,
.account-holder,
.note {
  font-size: 16px;
  color: #666;
  margin-top: 5px;
}
.label {
  font-weight: bold;
  color: #333;
}
#preview {
  visibility: hidden;
  border: 2px solid black;
  border-radius: 5px;
  width: 100%;
  height: 421px;
  background-repeat: no-repeat;
  background-size: 2105px 2105px;
  position: absolute;
  z-index: 999999;
  right: -105%;
  top: 0;
}
.bg-brown {
  background-color: #8b4513; /* Adjust color as needed */
}
.bg-gray-300 {
  background-color: #d3d3d3; /* Adjust color as needed */
}

#samples {
  text-align: center;
}

#samples img {
  display: block;
  border: 2px solid #6a6462;
}
.image {
  width: 100%;
  height: auto;
  object-fit: cover;
}
#samples img:hover {
  cursor: zoom-in;
  border: 0;
  -moz-box-shadow: 2px 2px 7px 2px rgba(130, 130, 130, 1),
    -1px -1px 7px 2px rgba(130, 130, 130, 1);
  -webkit-box-shadow: 2px 2px 7px 2px rgba(130, 130, 130, 0.7),
    -1px -1px 7px 2px rgba(130, 130, 130, 1);
  box-shadow: 2px 2px 7px 2px rgba(130, 130, 130, 0.7),
    -2px -2px 7px 2px rgba(130, 130, 130, 1);
}

@media screen and (max-width: 767px) {
  #samples img {
    display: inline-block;
  }
}
</style>
