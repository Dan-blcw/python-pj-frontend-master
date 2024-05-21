<!-- <script setup lang="ts">
import Product from "~/components/common/Product.vue";
</script>

<template>
  <h1>Account</h1>
</template>

<style lang="scss"></style> -->

account/index.vue:
<script setup lang="ts">
import { ref, onMounted } from "vue";
import { User, EditPen, Message } from "@element-plus/icons-vue";

// Reactive properties to hold form data
const avatarUrl = ref(
  "https://media.istockphoto.com/id/1341046662/vector/picture-profile-icon-human-or-people-sign-and-symbol-for-template-design.jpg?s=612x612&w=0&k=20&c=A7z3OK0fElK3tFntKObma-3a7PyO8_2xxW0jtmjzT78="
);
const firstname = ref("");
const lastname = ref("");
const username = ref("");
const email = ref("");
const userId = ref("");

//Xử lý thay đổi ảnh đại diện
const handleImageUpload = async (event: Event) => {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e: ProgressEvent<FileReader>) => {
      if (e.target) {
        avatarUrl.value = e.target.result as string;
        // Call API khi update ảnh
        updateAvatar(avatarUrl.value);
      }
    };
    reader.readAsDataURL(file);
  }
};

// Cập nhật Avatar
const updateAvatar = async (avatar: string) => {
  try {
    const response = await fetch("/api/update-avatar", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ avatar }),
    });
    const result = await response.json();
    if (result.success) {
      alert("Avatar updated successfully");
    }
  } catch (error) {
    console.error("Error updating avatar:", error);
  }
};

//Lấy thông tin người dùng
const fetchUserInfo = async () => {
  const user = localStorage.getItem("user") + "";
  const userPasre = JSON.parse(user);
  console.log(userPasre);

  username.value = userPasre.username;
  email.value = userPasre.email;
  firstname.value = userPasre.first_name;
  lastname.value = userPasre.last_name;
  userId.value = userPasre.id;

  // try {
  //   const accessToken = localStorage.getItem("accessToken");
  //   console.log({ accessToken });

  //   const response = await fetch("/api/user/3", {
  //     method: "GET",
  //     headers: {
  //       "Content-Type": "application/json",

  //       Authorization: `Bearer ${accessToken}`,
  //     },
  //   });
  //   const result = await response.json();
  //   if (result.success) {
  //     const { username, avatar, email, first_name, last_name } = result.data;
  //     console.log({ username, avatar, email, first_name, last_name });

  //     username.value = username;
  //     email.value = email;
  //     firstname.value = first_name;
  //     lastname.value = last_name;

  //     if (avatar) {
  //       avatarUrl.value = avatar;
  //     }
  //   }
  // } catch (error) {
  //   console.error("Error fetching user information:", error);
  // }
};

// Cập nhật thông tin người dùng
const updatePersonalInfo = async () => {
  console.log({
    first_name: firstname.value,
    last_name: lastname.value,
    username: username.value,
    email: email.value,
  });
  const user = localStorage.getItem("user") + "";
  const userPasre = JSON.parse(user);
  console.log(userPasre);

  try {
    const token = localStorage.getItem("accessToken") || '""';
    const response = await fetch(
      `http://127.0.0.1:8000/api/user/change-detail/${userId.value}/`,
      {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${JSON.parse(token)}`,
        },
        body: JSON.stringify({
          id: userPasre.id,
          is_superuser: userPasre.is_superuser,
          username: username.value,
          date_joined: userPasre.date_joined,
          is_active: userPasre.is_active,
          last_login: userPasre.last_login,
          email: email.value,
          first_name: firstname.value,
          last_name: lastname.value,
          cart: userPasre.cart,
        }),
      }
    );
    const result = await response.json();
    if (result.success) {
      alert("Personal information updated successfully");
    }
  } catch (error) {
    console.error("Error updating personal information:", error);
  }
};

// Đổi mật khẩu
const currentPassword = ref("");
const newPassword = ref("");
const retypeNewPassword = ref("");
const changePassword = async () => {
  if (newPassword.value !== retypeNewPassword.value) {
    alert("New passwords do not match");
    return;
  }

  try {
    const token = localStorage.getItem("accessToken") || '""';
    const urlChangepassword = `http://127.0.0.1:8000/api/user/change-password/${userId.value}/`;
    console.log(token);
    const response = await fetch(urlChangepassword, {
      method: "PATCH",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${JSON.parse(token)}`,
      },
      body: JSON.stringify({
        old_password: currentPassword.value,
        new_password: newPassword.value,
        renew_password: retypeNewPassword.value,
      }),
    });
    const result = await response.json();
    if (result.success) {
      alert("Password changed successfully");
    }
  } catch (error) {
    console.error("Error changing password:", error);
  }
};

//Lấy thông tin người dùng
onMounted(() => {
  fetchUserInfo();
});
</script>

<template>
  <div
    class="bg-slate-200 mt-[-15px] pt-2 bg-[url('https://theme.hstatic.net/1000026602/1001232314/14/img-banner-index.jpg?v=107')] bg-cover bg-center"
  >
    <div class="max-w-[1300px] mx-auto">
      <div class="mt-10 flex flex-row gap-10">
        <div
          class="basis-1/3 w-1/3 mb-20 bg-white rounded-md overflow-hidden h-fit"
        >
          <h2 class="text-xl bg-stone-900 px-4 py-2 text-white">
            Ảnh đại diện
          </h2>
          <div class="mx-auto rounded-full w-full my-4">
            <div
              class="relative w-40 h-40 mx-auto rounded-full flex justify-center overflow-hidden border-solid border-2 border-black"
            >
              <img
                :src="avatarUrl"
                class="w-full h-full object-cover"
                alt="No Avatar"
              />
              <label
                for="upload_img"
                class="absolute inset-0 bg-transparent cursor-pointer font-semibold text-transparent flex items-center justify-center hover:bg-black hover:opacity-50 hover:text-white"
              >
                <el-icon><EditPen /></el-icon>Đổi ảnh đại diện
              </label>
            </div>

            <form enctype="multipart/form-data">
              <p class="text-md w-full my-4 text-center">
                JPG or PNG no larger than 5 MB
              </p>
              <button
                class="block max-w-[200px] mx-auto px-4 py-2 bg-stone-900 hover:bg-white hover:text-black text-center text-lg text-white font-normal rounded-lg mt-5 cursor-pointer border-solid border-2 border-black transition-colors"
              >
                Cập nhật ảnh đại diện
              </button>
              <input
                type="file"
                name="upload_img"
                id="upload_img"
                accept="image/*"
                @change="handleImageUpload"
                hidden
              />
            </form>
          </div>
          <div class="p-4 border-t-4 border-solid border-slate-200">
            <div class="flex gap-2 text-lg mb-2">
              <span class="min-w-[150px] font-bold flex items-center gap-2"
                >First Name:</span
              >
              <p>{{ firstname }}</p>
            </div>
            <div class="flex gap-2 text-lg mb-2">
              <span class="min-w-[150px] font-bold flex items-center gap-2"
                >Last Name:</span
              >
              <p>{{ lastname }}</p>
            </div>
            <div class="flex gap-2 text-lg mb-2">
              <span class="min-w-[150px] font-bold flex items-center gap-2"
                >Email:</span
              >
              <p>{{ email }}</p>
            </div>
          </div>
        </div>
        <div class="basis-2/3 w-2/3 mb-20 h-fit">
          <div class="mb-10 rounded-md overflow-hidden">
            <h2 class="text-xl bg-stone-900 px-4 py-2 text-white">
              Thông tin chi tiết
            </h2>
            <form
              name="details_info"
              class="p-4 flex flex-wrap flex-row gap-2 justify-between bg-white"
              @submit.prevent="updatePersonalInfo"
            >
              <div class="basis-[48%] w-[48%]">
                <label for="username">Tên đăng nhập</label>
                <input
                  type="text"
                  id="username"
                  v-model="username"
                  placeholder="Username..."
                  class="border-2 border-solid border-[#ccc] px-4 py-2 w-full rounded-lg focus:outline-2 focus:outline-black"
                  readonly
                />
              </div>
              <div class="basis-[48%] w-[48%]">
                <label for="email">Email</label>
                <input
                  type="email"
                  id="email"
                  v-model="email"
                  placeholder="Email..."
                  class="border-2 border-solid border-[#ccc] px-4 py-2 w-full rounded-lg focus:outline-2 focus:outline-black"
                />
              </div>

              <div class="basis-[48%] w-[48%]">
                <label for="firstname">First name</label>
                <input
                  type="text"
                  id="firstname"
                  v-model="firstname"
                  placeholder="Firstname..."
                  class="border-2 border-solid border-[#ccc] px-4 py-2 w-full rounded-lg focus:outline-2 focus:outline-black"
                />
              </div>

              <div class="basis-[48%] w-[48%]">
                <label for="lastname">Last name</label>
                <input
                  type="text"
                  id="lastname"
                  v-model="lastname"
                  placeholder="Lastname..."
                  class="border-2 border-solid border-[#ccc] px-4 py-2 w-full rounded-lg focus:outline-2 focus:outline-black"
                />
              </div>

              <div class="flex justify-end basis-full w-full">
                <button
                  type="submit"
                  class="px-4 py-2 bg-stone-900 text-white rounded-lg mt-4 border-2 border-solid border-black hover:text-teal-700 hover:bg-white transition-colors"
                >
                  Lưu thay đổi
                </button>
              </div>
            </form>
          </div>
          <div class="rounded-md overflow-hidden">
            <h2 class="text-xl bg-stone-900 px-4 py-2 text-white">
              Thay đổi mật khẩu
            </h2>
            <form
              name="change_password"
              class="p-4 flex flex-wrap flex-row gap-2 justify-between bg-white"
              @submit.prevent="changePassword"
            >
              <div class="basis-[48%] w-[48%]">
                <label for="current_password">Mật khẩu hiện tại</label>
                <input
                  type="password"
                  id="current_password"
                  v-model="currentPassword"
                  placeholder="Nhập mật khẩu hiện tại"
                  class="border-2 border-solid border-[#ccc] px-4 py-2 w-full rounded-lg focus:outline-2 focus:outline-black"
                />
              </div>
              <div class="basis-[48%] w-[48%]">
                <label for="new_password">Mật khẩu mới</label>
                <input
                  type="password"
                  id="new_password"
                  v-model="newPassword"
                  placeholder="Nhập mật khẩu mới"
                  class="border-2 border-solid border-[#ccc] px-4 py-2 w-full rounded-lg focus:outline-2 focus:outline-black"
                />
              </div>
              <div class="basis-[48%] w-[48%]">
                <label for="retype_new_password">Nhập lại mật khẩu mới</label>
                <input
                  type="password"
                  id="retype_new_password"
                  v-model="retypeNewPassword"
                  placeholder="Nhập lại mật khẩu mới"
                  class="border-2 border-solid border-[#ccc] px-4 py-2 w-full rounded-lg focus:outline-2 focus:outline-black"
                />
              </div>
              <div class="flex justify-end basis-full w-full">
                <button
                  type="submit"
                  class="px-4 py-2 bg-stone-900 text-white rounded-lg mt-4 border-2 border-solid border-black hover:text-teal-700 hover:bg-white transition-colors"
                >
                  Lưu thay đổi
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss"></style>
