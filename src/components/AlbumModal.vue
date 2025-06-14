<script setup>
import { onMounted, ref } from "vue";
import { useModalStore } from "@/stores/modal";
import { useAuthStore } from '@/stores/auth';
import { Icon } from '@iconify/vue';
import { storeToRefs } from "pinia";
import axios from 'axios';
import defaultImgage from '@/assets/default.jpg'
import { useActivityStore } from "@/stores/activity";
import { useViewStore } from "@/stores/view";


const authStore = useAuthStore();
const useModal = useModalStore()
const useView = useViewStore();
const { openEditAlbum, playlistEditData } = storeToRefs(useModal)
const useActivity = useActivityStore();

const previewImg = ref(null)
const fileInput = ref(null)


const albumName = ref(playlistEditData.value.name);
const albumDescrip = ref(playlistEditData.value.description);
const albumPrice = ref(playlistEditData.value.price);

const thumbnail = ref(null)


function onImgChoosed(event) {
    const file = event.target.files[0]
    if (file) {
        thumbnail.value = file
        previewImg.value = URL.createObjectURL(file)
    }
}
function chooseImg() {
    fileInput.value?.click()
}


const saveProfile = async () => {
    if (!albumName.value || !albumDescrip.value || !albumPrice.value) {
        useActivity.addNotify(true, "Vui lòng điền đầy đủ thông tin!")
        return;
    }
    if(albumPrice.value < 4000){
        useActivity.addNotify(true, "Giá Album tối thiểu là 4000đ!")
        return;
    }
    const formData = new FormData()
    formData.append('name', albumName.value)
    formData.append('description', albumDescrip.value)
    formData.append('price', albumPrice.value)
    if (thumbnail.value) {
        formData.append('thumbnail', thumbnail.value)
    }

    try {
        const res = await axios.post(`http://spotify_clone_api.test/api/profile/update-album/${playlistEditData.value.id}`, formData, {
            'headers': {
                'Authorization': 'Bearer ' + authStore.user.token,
            },
        });
        if (res.data.code == 200) {
            useActivity.fetchData();
            useActivity.fetchUserData();
            useActivity.addNotify(false, "Chỉnh sửa thành công!")
            useActivity.syncGdrive('thumbnails', playlistEditData.value.id)
        }
    } catch (e) {
        console.log(e);
        useActivity.addNotify(true, "Chỉnh sửa thất bại!")
    }
    openEditAlbum.value = false
};

onMounted(() => {
    console.log(playlistEditData.value)
})
</script>
<template>
    <div class="fixed inset-0 z-[100] flex items-center justify-center bg-black bg-opacity-60">
        <div class="bg-[#1D1512] text-white rounded-lg w-[700px] p-6 relative">
            <button class="absolute text-white top-4 right-4 hover:text-red-500" @click="openEditAlbum = false">
                ✕
            </button>

            <h2 class="mb-6 text-2xl font-bold" :style="{color: useView.currentColor}">Chỉnh sửa Album</h2>

            <div class="flex gap-1 mb-3">
                <div
                    class="relative flex items-center justify-center w-64 h-64 cursor-pointer rounded-xl bg-zinc-700 shrink-0 group">
                    <img class="object-cover w-full h-full rounded-xl "
                        :src="previewImg ? previewImg : playlistEditData.thumbnail_path"
                        @error="event => event.target.src = defaultImgage" alt="">
                    <div class="absolute inset-0 flex items-center justify-center transition bg-black bg-opacity-50 opacity-0 rounded-xl group-hover:opacity-100"
                        @click="chooseImg">
                        <Icon icon="material-symbols:edit-rounded text-black " />
                        <input type="file" accept="image/*" ref="fileInput" style="display: none"
                            @change="onImgChoosed" />
                    </div>
                </div>
                <div class="flex-1 ml-2">
                    <input v-model="albumName" class="px-4 py-3 mb-3 text-white rounded-lg bg-[#25211F] w-full"
                        placeholder="Tên Album" />

                    <textarea v-model="albumDescrip" rows="5" class="px-4 py-3 mb-3 text-white rounded-lg bg-[#25211F] w-full"
                        placeholder="Mô tả Album..."></textarea>

                    <input v-model="albumPrice" type="number"
                        class="px-4 py-3 mb-3 text-white rounded-lg bg-[#25211F] w-full" placeholder="Giá" />
                </div>
            </div>

            <div class=" float-end">
                <button class="py-2 font-semibold text-white transition bg-gray-600 px-7 rounded-3xl hover:bg-gray-500"
                    @click="openEditAlbum = false">
                    Hủy
                </button>
                <button class="px-7 py-2 ml-4 font-semibold text-white transition bg-[#BC4D15] rounded-3xl hover:brightness-125"
                    :style="{backgroundColor: useView.currentColor}"
                    @click="saveProfile">
                    Lưu
                </button>
            </div>
            <p class="mt-16 text-xs text-gray-400">
                Bằng cách tiếp tục, bạn đồng ý cho phép TÔI truy cập vào hình ảnh
                bạn đã chọn để tải lên. Vui lòng đảm bảo bạn có quyền tải lên hình ảnh.
            </p>
        </div>
    </div>
</template>
