<script setup>
import { ref } from 'vue'
import {ShoppingCart} from "@element-plus/icons-vue";
import {fakeProducts} from "@/constants/fakeData.js"
import spaFetch from "~/plugins/fetch.js";


const { $apiUrl } = useNuxtApp()

const products = ref(fakeProducts)
const router = useRouter()
const value = ref('Option1')
const loading = ref(false)

const formState = reactive({
    page: 1,
    size: 20,
    total: 0
})

const getProducts = () =>{
    loading.value = true
    spaFetch(false)($apiUrl.PRODUCT,{
        method: 'GET',
        params:{
            page: formState.page,
            pageSize: formState.size,
            search: 'gundam',
        },

    }).then( res => {
        products.value = res.results.map(item =>{
            return {
                ...item,
                status: item.type
            }
        })
        loading.value = false
        formState.total = res.count
    }).catch( error => {
        loading.value = false
        console.log("error", error.response)
    })
}

const advertising =
    {
        img1:'https://theme.hstatic.net/1000026602/1001232314/14/slideshow_2.jpg?v=107',
        img2:'https://i.pinimg.com/564x/e5/de/e0/e5dee07399cece6e23146ad9af1554f5.jpg',
    }


const options = [
    {
        value: 'Option1',
        label: 'Mặc định',
    },
    {
        value: 'Option2',
        label: 'Tên A-Z',
    },
    {
        value: 'Option3',
        label: 'Tên Z-A',
    },
    {
        value: 'Option4',
        label: 'Giá giảm dần',
    },
    {
        value: 'Option5',
        label: 'Giá tăng dần',
    },
]

const handleClick = (item) => {
    router.push(item && item.id ? `/product/${item.id}` : '')
}

const handleChangePage = (value) => {
    products.value = []
    getProducts()
}

getProducts()
</script>

<template>
    <div>
       <div v-loading="loading" class="w-4/5 flex flex-col items-center justify-center  mx-auto">
           <!-- <CategoryProducts> -->
               <img :src="advertising && advertising.img1 ? advertising.img1 : ''"
                    alt="" sizes="700px">
               <!-- <div class="flex items-center justify-center p-3 bg-red-600 my-4 teẬT Bxt-white font-semibold">
                   T1 SHOP GUNDAM - CHUYÊN MUA BÁN MÔ HÌNH GUNDAM GUNPLA CHÍNH HÃNG BANDAI NHẢN
               </div> -->
               <!-- <img :src="advertising && advertising.img2 ? advertising.img2 : ''" alt=""> -->
                <!-- <div class="flex h-16 items-center bg-gray-500 px-8">
                    <div>
                        <span class="text-white">
                        Sắp xếp theo:
                        </span>
                        <el-select
                            v-model="value"
                            class="m-2"
                            default-first-option
                            size="small"
                            style="width: 120px;"
                        >
                            <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value"
                            />
                        </el-select>
                    </div>
                </div> -->
               <div class="w-3/4 grid grid-cols-4 gap-4">
                   <div v-for="(product, index) in products"
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
                   </div>
               </div>
               <el-pagination
                   v-model:current-page="formState.page"
                   :page-size="formState.size"
                   layout="prev, pager, next"
                   :total="formState.total"
                   @currentChange="handleChangePage"
               />
           <!-- </CategoryProducts> -->
       </div>
    </div>
</template>


<style scoped>
  .card-new-product {
    transition: all 0.3s;
  }
</style>

