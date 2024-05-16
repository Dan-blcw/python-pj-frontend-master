<script setup>
import {Monitor, ShoppingCart} from "@element-plus/icons-vue";
import {ref} from 'vue'
import {Swiper, SwiperSlide} from "swiper/vue";
import {Autoplay, Navigation} from "swiper/modules";
import {fakeProducts} from "~/constants/fakeData.js";
import spaFetch from "~/plugins/fetch.js";
import {useAuthStore} from "~/stores/auth.js";


const {$apiUrl} = useNuxtApp()
const route = useRoute()
const router = useRouter()
const selectedSize = ""
const id = route.params ? route.params.id : null

const modules = ref([Autoplay, Navigation])
const products = ref(fakeProducts)
const dataDetail = ref(null)
const loading = ref(false)
const loadingAdd = ref(false)
const auth = useAuthStore()

const num = ref(1)
const textarea2 = ref('')
const activeName = ref('first')
// const detailInfo = ref(
//     {
//         name: 'Mô hình Gundam MG 1/100 ASW-G-08 Gundam Barbatos - Bandai - GDMG0004',
//         description: '1 2 3 test mic alo alo',
//         price: '1 ti dong :)))',
//         type: 'Gundam',
//         detailIMg: [
//             {
//                 img: 'https://herogame.vn/upload/images/img_10_04_2023/mo-hinh-gundam-mg-1-100-asw-g-08-gundam-barbatos-bandai-gdmg0004-1_434072_6433dfd319b4d1.71893000.jpg',
//             },
//             {
//                 img: 'https://herogame.vn/upload/images/img_10_04_2023/mo-hinh-gundam-mg-1-100-asw-g-08-gundam-barbatos-bandai-gdmg0004-1_434072_6433dfd319b4d1.71893000.jpg',
//             },
//             {
//                 img: 'https://herogame.vn/upload/images/img_10_04_2023/mo-hinh-gundam-mg-1-100-asw-g-08-gundam-barbatos-bandai-gdmg0004-1_434072_6433dfd319b4d1.71893000.jpg',
//             },
//             {
//                 img: 'https://herogame.vn/upload/images/img_10_04_2023/mo-hinh-gundam-mg-1-100-asw-g-08-gundam-barbatos-bandai-gdmg0004-1_434072_6433dfd319b4d1.71893000.jpg',
//             },
//             {
//                 img: 'https://herogame.vn/upload/images/img_10_04_2023/mo-hinh-gundam-mg-1-100-asw-g-08-gundam-barbatos-bandai-gdmg0004-1_434072_6433dfd319b4d1.71893000.jpg',
//             },
//             {
//                 img: 'https://herogame.vn/upload/images/img_10_04_2023/mo-hinh-gundam-mg-1-100-asw-g-08-gundam-barbatos-bandai-gdmg0004-1_434072_6433dfd319b4d1.71893000.jpg',
//             },
//         ],
//     }
// )
// const zoomImg = () => {
//     // console.log($("#zoom_01"))
//     // $("#zoom_01").ezPlus()
// }

const getProducts = () => {
    spaFetch(false)($apiUrl.PRODUCT, {
        method: 'GET',
        params: {
            page: 3,
            pageSize: 10,
        }
    }).then(res => {
        products.value = res.results.map(item => {
            return {
                ...item,
                status: item.type
            }
        })
    }).catch(error => {
        console.log("error", error.response)
    })
}
const getDetail = () => {
    loading.value = true
    spaFetch(false)(`${$apiUrl.PRODUCT}${id}/`, {
        method: 'GET'
    }).then(res => {

        dataDetail.value = res
        loading.value = false
    }).catch(error => {
        console.log("error", error.response)
        loading.value = false
    })
}

const zoomIn = (event) => {
    const pre = document.getElementById("preview");
    pre.style.visibility = "visible";
    if ($('#zoom1').is(':hover')) {
        pre.style.backgroundImage = `url('${dataDetail.value && dataDetail.value.img ? dataDetail.value.img : null}')`;
    }
    const posX = event.offsetX;
    const posY = event.offsetY;
    pre.style.backgroundPosition = (-posX * 3.8) + "px " + (-posY * 3.5) + "px";
}

const zoomOut = () => {
    const pre = document.getElementById("preview");
    pre.style.visibility = "hidden";
}

const handleAddProductToCart = () => {
    if (!auth.$state.accessToken || !auth.$state.refreshToken) {
        return router.push('/login')
    }
    if (loadingAdd.value) {
        return
    }
    loadingAdd.value = true
    spaFetch()($apiUrl.CART_ITEM, {
        method: 'POST',
        body: {
            "product": id,
            "quantity": num.value,
        }
    }).then(res => {
        ElMessage.success(
            "Thêm vào giỏ hàng thành công"
        )
        loadingAdd.value = false
        getUser()
    }).catch(error => {
        console.log("error", error.response)
        if (error.status === 400) {
            ElMessage.error(
                {
                    message: "Số lượng hàng trong kho không đủ",
                }
            )
        } else {
            ElMessage.error(
                "Thêm vào giỏ hàng thất bại"
            )
        }
        loadingAdd.value = false
    })
}

const handleChange = (value, oldValue) => {
    if (!value) {
        return num.value = oldValue
    }
}
const getUser = () => {
    const idUser = auth.$state.user && auth.$state.user.id ? auth.$state.user.id : null
    spaFetch()(`${$apiUrl.USER}${idUser}/`, {
        method: 'GET'
    }).then(res => {
        auth.setUser(res)
        getCart()
    }).catch(error => {
        console.log("error", error.response)
    })
}

const getCart = () => {
    const id = auth.$state.user && auth.$state.user.cart ? auth.$state.user.cart : null
    spaFetch()(`${$apiUrl.CART}${id}/`, {
        method: 'GET'
    }).then(res => {
        auth.setQuantityInCart(res.products.length)
    }).catch(error => {
        console.log("error", error.response)
    })
}

getCart()
getDetail()
getProducts()
</script>

<template>
    <div class="flex flex-col">
        <div class="w-full flex flex-col items-center justify-center  mx-auto">

            <div v-loading="loading" class="w-full flex mt-4 justify-center ">
                <div class="flex flex-col items-center">
                    <div class="w-[600px] h-[600px] bg-white flex items-center justify-center">
                        <div id="samples" class="relative">
                            <img id="zoom1"
                                :src="dataDetail && dataDetail.img ? dataDetail.img : ''"
                                alt=""
                                @mousemove="zoomIn"
                                @mouseout="zoomOut">
                            <div id="preview"></div>
                        </div>
                    </div>
                </div>

                <div class="ml-9 flex flex-col gap-y-2 w-[450px]">
                    <div class="text-[1.85rem] text-[#0E7069] font-semibold">
                        {{ dataDetail && dataDetail.name ? dataDetail.name : '' }}
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
                        {{ dataDetail && dataDetail.description ? dataDetail.description : '' }}
                    </div>

                    <div class="flex text-black text-lg items-baseline text-[1.00rem] font-semibold">
                        Giá tiền :
                        <div class="font-semibold text-teal-700 text-lg ml-4">
                            {{ dataDetail && dataDetail.price ? dataDetail.price : 0 }} VND
                        </div>
                    </div>

                    <div class="flex text-black text-lg items-baseline text-[1.00rem] font-semibold">
                        Trạng thái :
                        <div class="font-semibold text-amber-700 text-lg ml-4 uppercase"> Còn Hàng</div>
                    </div>

                    <div class="flex text-black text-lg items-baseline text-[1.00rem] font-semibold">
                        Bộ sưu tầm :
                        <div class="font-semibold text-teal-700 text-lg ml-4 underline cursor-pointer">
                            {{ dataDetail && dataDetail.type ? dataDetail.type : '' }}
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
                    <div class="flex flex-col gap-y-1 mt-2">
                        <div class="text-black text-lg  text-[1.05rem] font-semibold">Chọn màu sắc:</div>
                        <div class="flex gap-x-4">
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedColor" value="nâu">
                                <span class="w-6 h-6 rounded-full bg-brown inline-block"></span> Nâu
                            </label>
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedColor" value="đen">
                                <span class="w-6 h-6 rounded-full bg-black inline-block"></span> Đen
                            </label>
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedColor" value="trắng xám">
                                <span class="w-6 h-6 rounded-full bg-gray-300 inline-block"></span> Trắng Xám
                            </label>
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedColor" value="Xanh">
                                <span class="w-6 h-6 rounded-full bg-sky-700 inline-block"></span> Xanh
                            </label>
                        </div>
                    </div>

                    <!-- Size Selection -->
                    <div class="flex flex-col gap-y-1 mt-4">
                        <div class="text-black text-lg text-[1.05rem] font-semibold">Chọn kích thước:</div>
                        <div class="flex gap-x-4">
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedSize" value="S" @change="handleSizeChange"> S
                            </label>
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedSize" value="M" @change="handleSizeChange"> M
                            </label>
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedSize" value="L" @change="handleSizeChange"> L
                            </label>
                            <label class="flex items-center gap-x-1">
                                <input type="radio" v-model="selectedSize" value="XL" @change="handleSizeChange"> XL
                            </label>
                        </div>
                    </div>
        <div>
    <!-- Áo Size Chart -->
                <div class="text-black text-lg text-[1.05rem] font-semibold">Hướng dẫn cách đo kích thước sản phẩm:</div>
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
                    <p>*Dữ liệu này có được bằng cách đo thủ công sản phẩm, các phép đo có thể bị thay đổi 1-2 CM.</p>
                </div>

        <div class="flex my-3 gap-x-32 items-center translate-x-24">
            <el-input-number class="h-8" v-model="num" size="large" :min="1" @change="handleChange"/>
            
        </div>
        <div class="flex w-3/4 h-12 bg-gray-800 items-center translate-x-4 justify-center px-10 gap-x-4 hover:bg-teal-700 cursor-pointer"
                 @click="handleAddProductToCart" v-loading="loadingAdd">
                <el-icon color="white" size="24">
                    <ShoppingCart/>
                </el-icon>
                <p class="uppercase text-white text-base">Thêm vào giỏ</p>
        </div>
    </div>
</div>

            <!-- <div v-loading="loading" class="w-full flex mt-10 justify-center">
                <div class="flex flex-col items-center">
                    <div class="w-[450px] h-[650px] bg-gray-400 flex items-center justify-center">
                        <div id="samples" class="relative">
                            <img id="zoom1"
                                :src="dataDetail && dataDetail.img ? dataDetail.img : ''"
                                alt=""
                                @mousemove="zoomIn"
                                @mouseout="zoomOut">
                            <div id="preview"></div>
                        </div>
                    </div>
                </div>

                <div class="ml-20 flex flex-col gap-y-3 w-[600px]">
                    <div class="text-[1.85rem] text-[#17649a] font-semibold">
                        {{ dataDetail && dataDetail.name ? dataDetail.name : '' }}
                    </div>

                    <div class="text-black italic">
                        {{ dataDetail && dataDetail.description ? dataDetail.description : '' }}
                    </div>

                    <div class="flex text-black text-lg items-baseline">
                        Giá tiền :
                        <div class="font-semibold text-red-500 text-2xl ml-10">
                            {{ dataDetail && dataDetail.price ? dataDetail.price : 0 }} đ
                        </div>
                    </div>

                    <div class="flex text-black text-lg items-baseline">
                        Trạng thái :
                        <div class="font-semibold text-green-500 text-lg ml-4 uppercase"> Còn Hàng</div>
                    </div>

                    <div class="flex text-black text-lg items-baseline">
                        Danh mục :
                        <div class="font-semibold text-blue-700 text-lg ml-4 underline hover:text-blue-500 cursor-pointer">
                            {{ dataDetail && dataDetail.type ? dataDetail.type : '' }}
                        </div>
                    </div>

                    <div class="flex text-black text-lg items-baseline flex-wrap gap-y-1">
                        Từ khoá :
                        <div class="font-semibold text-white text-lg ml-4 bg-blue-600 px-2 rounded-lg hover:bg-blue-500">
                            {{ dataDetail && dataDetail.type ? dataDetail.type : '' }}
                        </div>
                        <div class="font-semibold text-white text-lg ml-4 bg-blue-600 px-2 rounded-lg hover:bg-blue-500">
                            {{ dataDetail && dataDetail.name ? dataDetail.name : '' }}
                        </div>
                    </div>

                    <div class="flex my-10 gap-x-32 items-center">
                        <el-input-number class="h-20" v-model="num" size="large" :min="1" @change="handleChange"/>
                        <div class="flex h-20 bg-red-600 items-center justify-center px-10 gap-x-6 hover:bg-red-500 cursor-pointer"
                            @click="handleAddProductToCart" v-loading="loadingAdd">
                            <el-icon color="white" size="40">
                                <ShoppingCart/>
                            </el-icon>
                            <p class="uppercase text-white text-2xl">Thêm vào giỏ</p>
                        </div>
                    </div>
                </div>
            </div> -->


                <!-- <div class="w-1/2 mt-">
                    <el-tabs
                        v-model="activeName"
                        type="border-card"
                        class="demo-tabs"
                        stretch
                    >
                        <el-tab-pane label="Đánh giá" name="first">
                            <div class="flex justify-between mb-4">
                                <p>0 bình luận</p>
                                <p>Sắp xếp theo</p>
                            </div>
                            <el-input
                                v-model="textarea2"
                                :autosize="{ minRows: 2, maxRows: 4 }"
                                type="textarea"
                                placeholder="Nhập bình luận ..."
                            />
                            <div class="mt-4 text-right">
                                <el-button type="primary" plain>Đăng</el-button>
                            </div>
                        </el-tab-pane>
                        <el-tab-pane label="Quyền lợi" name="second">
                            <el-descriptions :column="1" title="Được gì khi mua máy tại HEROGAME ?">
                                <el-descriptions-item>- Giảm giá 100k khi mua MÁY lần sau</el-descriptions-item>
                                <el-descriptions-item>- Phụ kiện zin theo máy</el-descriptions-item>
                                <el-descriptions-item>- Đổi máy mới trong 7 ngày khi máy gặp lỗi của NSX, hoàn tiền 100%
                                    khi không còn máy đổi
                                </el-descriptions-item>
                                <el-descriptions-item>- Tích lũy điểm giảm giá cho những đơn hàng sau
                                </el-descriptions-item>
                                <el-descriptions-item>- Bảo hành phần mềm trong suốt quá trình sử dụng
                                </el-descriptions-item>
                                <el-descriptions-item>- Hỗ trợ sửa chữa cho khách hàng khi máy hết BH
                                </el-descriptions-item>
                                <el-descriptions-item>- Hỗ trợ đổi máy/ lên đời khi vẫn còn BH</el-descriptions-item>
                                <el-descriptions-item>- Cập nhập Firmware mới liên tục cho các dòng máy
                                </el-descriptions-item>
                            </el-descriptions>
                        </el-tab-pane>
                        <el-tab-pane label="Thanh toán" name="third">
                            <el-descriptions :column="1" title="Thanh toán">
                                <el-descriptions-item>- Áp dụng thanh toán quẹt thẻ ATM/VISA/MASTER
                                </el-descriptions-item>
                                <el-descriptions-item>- Có thanh toán chuyển khoản online / Momo</el-descriptions-item>
                            </el-descriptions>
                            <el-descriptions :column="1" title="">
                                <el-descriptions-item><span class="text-[#ff0000] font-semibold text-xl ml-16">CHI NHÁNH HỒ CHÍ MINH</span>
                                </el-descriptions-item>
                                <el-descriptions-item><span
                                    class="text-black font-semibold">NGÂN HÀNG VIETCOMBANK</span></el-descriptions-item>
                                <el-descriptions-item label="Số tài khoản:"><span class="text-black font-semibold">83860101002</span>
                                </el-descriptions-item>
                                <el-descriptions-item label="Chủ tài khoản:"><span class="text-black font-semibold">NGUYỄN KỲ NAM</span>
                                </el-descriptions-item>
                            </el-descriptions>
                            <el-descriptions :column="1" title="">
                                <el-descriptions-item><span class="text-[#ff0000] font-semibold text-xl ml-16">CHI NHÁNH HÀ NỘI</span>
                                </el-descriptions-item>
                                <el-descriptions-item><span class="text-black font-semibold">NGÂN HÀNG VIETCOMBANK (Chi nhánh: Tây Hà Nội)</span>
                                </el-descriptions-item>
                                <el-descriptions-item label="Số tài khoản:"><span class="text-black font-semibold">83860101002</span>
                                </el-descriptions-item>
                                <el-descriptions-item label="Chủ tài khoản:"><span class="text-black font-semibold">NGUYỄN KỲ NAM</span>
                                </el-descriptions-item>
                                <el-descriptions-item><span class="text-[#ff0000] font-semibold text-lg">* Lưu ý : Chuyển khoản với cú pháp là : "Số điện thoại" - Sau khi chuyển khoản vui lòng liên hệ Zalo hoặc fan page để xác nhận.</span>
                                </el-descriptions-item>
                            </el-descriptions>
                        </el-tab-pane>
                    </el-tabs>
                </div> -->
                <div class="w-full mt-12 ">
                    <div
                        class=" w-2/5 bg-cyan-700 translate-x-[75%] flex h-[45px] justify-center items-center text-white font-semibold text-lg rounded-xl">
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
                        class="w-3/5 "

                    >
                        <swiper-slide v-for="(product, index) in products"
                            :key="index"
                            class="mt-4 hover:mt-1 card-new-product">
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
.demo-tabs > .el-tabs__content {
    padding: 32px;
    color: #6b778c;
    font-size: 15px;
    font-weight: 600;
}

.demo-tabs .el-tabs__item {
    font-size: 18px;
    font-weight: 300;
}

.el-descriptions__title {
    color: red;
}

table {
  width: 100%;
  border-collapse: collapse;
}

table, th, td {
  border: 1px solid black;
}

th, td {
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
    background-color: #8B4513; /* Adjust color as needed */
}
.bg-gray-300 {
    background-color: #D3D3D3; /* Adjust color as needed */
}

#samples {
    text-align: center;
}

#samples img {
    display: block;
    border: 2px solid #6A6462;
}

#samples img:hover {
    cursor: zoom-in;
    border: 0;
    -moz-box-shadow: 2px 2px 7px 2px rgba(130, 130, 130, 1), -1px -1px 7px 2px rgba(130, 130, 130, 1);
    -webkit-box-shadow: 2px 2px 7px 2px rgba(130, 130, 130, .7), -1px -1px 7px 2px rgba(130, 130, 130, 1);
    box-shadow: 2px 2px 7px 2px rgba(130, 130, 130, .7), -2px -2px 7px 2px rgba(130, 130, 130, 1);
}

@media screen and (max-width: 767px) {
    #samples img {
        display: inline-block;
    }
}


</style>