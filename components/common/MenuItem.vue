<script setup>
import SubMenuItem from "~/components/common/SubMenuItem.vue";

const props = defineProps({
  item: {
    type: Object,
    default() {
      return null;
    },
  },
});
 
const router = useRouter();
 
const getMainSubMenu = () => {
  if (props.item && props.item.mainSubMenu) {
    return props.item.mainSubMenu;
  }
  return "";
};
 
const handleClick = (item) => {
  router.push(item && item.link ? item.link : "#");
};
</script>
 
<template>
  <div>
    <div
      class="flex cursor-pointer justify-center items-center menu-item-title mx-4"
      @click="handleClick(item)"
    >
      {{ item && item.title ? item.title : "" }}
    </div>
  <!-- <div
    class="relative h-[calc(100vh-90px)] mb-10 py-10 bg-[url('https://wallpaper.dog/large/20413086.jpg')] bg-cover bg-no-repeat bg-center bg-fixed"
  > -->
    <div
      v-if="item.isSubmenu"
      class="menu-item-drop w-3/5 h-[550px] absolute left-1/2 translate-x-[-50%] bg-[url('https://i.pinimg.com/564x/b3/74/d5/b374d5de39f1c664bfc6c258ccb22566.jpg')] justify-center items-center flex invisible opacity-0 overflow-hidden"
      :class="item && item.colorBg ? item.colorBg : ''"
    >
      <div class="flex flex-row items-start">

        <div
          v-if="item && item.children && item.children.length > 0"
          class="grid grid-cols-4 grid-rows-3 gap-4 justify-between mr-4"
        >
          <SubMenuItem
            v-for="(i, index) in item.children"
            :key="index"
            :data="i"
            @click="handleClick(i)"
            :color-item-s-m="item && item.colorItemSM ? item.colorItemSM : ''"
          />
        </div>

        <div
          class="w-60 h-96 rounded-2xl mr-4 menu-item-image overflow-hidden cursor-pointer"
          :class="item && item.colorItemSM ? item.colorItemSM : ''"
          @click="handleClick(item)"
        >
          <div
            class="w-full h-full"
            :style="`background-image: url(${getMainSubMenu().img})`"
          ></div>
          <span class="text-black text-xl">{{ getMainSubMenu().title }}</span>
        </div>
 
      </div>
    </div>
  </div>
</template>
 
<style scoped>
.menu-item-drop{
  border-radius: 20px;
}
</style>
<!-- <script setup>
import SubMenuItem from "~/components/common/SubMenuItem.vue";

const props = defineProps({
    item: {
        type: Object,
        default() {
            return null
        }
    }
})

const router = useRouter()

const getMainSubMenu = () => {
    if (props.item && props.item.mainSubMenu) {
        return props.item.mainSubMenu
    }
    return '';
}

const handleClick = (item) => {
    router.push(item && item.link ? item.link : "#")
}
</script>

<template>
    <div>
        <div class="w-[240px] flex cursor-pointer justify-center items-center menu-item-title" @click="handleClick(item)">
            {{ item && item.title ? item.title : '' }}
        </div>

        <div v-if="item.isSubmenu"
             class="menu-item-drop w-screen h-[600px] absolute left-0 justify-center items-center flex invisible opacity-0 overflow-hidden"
             :class="item && item.colorBg ? item.colorBg : ''">
            <div class="flex flex-row  items-start">
                <div
                    class="w-60 h-96 rounded-2xl mr-4 menu-item-image overflow-hidden cursor-pointer"
                    :class="item && item.colorItemSM ? item.colorItemSM : ''"
                    @click="handleClick(item)">
                    <div class="w-full h-full " :style="`background-image: url(${getMainSubMenu().img})`"></div>
                    <span class="text-white text-2xl">{{ getMainSubMenu().title }}</span>
                </div>

                <div v-if="item && item.children && item.children.length > 0" class="grid grid-cols-3 grid-rows-2 gap-4 justify-between ">
                    <SubMenuItem v-for="(i, index) in item.children"
                                 :key="index"
                                 :data="i"
                                 @click="handleClick(i)"
                                 :color-item-s-m="item && item.colorItemSM ? item.colorItemSM : ''"/>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>

</style> -->