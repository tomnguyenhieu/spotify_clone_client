<script setup>
import { ref, computed, onMounted } from "vue";
import { Icon } from '@iconify/vue';
import { useViewStore } from "@/stores/view";
import { storeToRefs } from "pinia";
import { useSongStore } from "@/stores/song";
import { useAuthStore } from '@/stores/auth';
import { useModalStore } from "@/stores/modal";
import { useActivityStore } from "@/stores/activity";
import axios from 'axios';
import defaultImgage from '@/assets/default.jpg'

let openMenu = ref(false);
let openUploadAlbum = ref(false);

const useActivity = useActivityStore();
const useView = useViewStore();
const authStore = useAuthStore();
const useSong = useSongStore();
const useModal = useModalStore();
const { currentComponent, isFullscreen } = storeToRefs(useView)
const { openEditProfile, openUploadSong, openEditAlbum } = storeToRefs(useModal)
const { myPlaylistList, followArtistList, myAlbumList, mySongList } = storeToRefs(useActivity)

const bills = ref([])

async function getBills() {
    try {
        const res = await axios.get(`http://spotify_clone_api.test/api/profile/payment-history`, {
            'headers': {
                'Authorization': 'Bearer ' + authStore.user.token,
            }
        });
        console.log(res.data.data)
        if (res.data.code == 200) {
            bills.value = res.data.data
        }
    } catch (e) {
        console.log(e);
        useActivity.addNotify(true, "Call Api thất bại!")
    }
}


async function createAlbum() {
    try {
        const res = await axios.get(`http://spotify_clone_api.test/api/profile/create-album`, {
            'headers': {
                'Authorization': 'Bearer ' + authStore.user.token,
            }
        });
        if (res.data.code == 200) {
            useActivity.addNotify(false, "Tạo Album thành công!")
            useActivity.fetchUserData();
        }
    } catch (e) {
        console.log(e);
        useActivity.addNotify(true, "Call Api thất bại!")
    }
}

function albumCheck(){
    if(myAlbumList.value.length > 0){
        openUploadSong.value = true
    }
    else{
        useActivity.addNotify(true, "Vui lòng tạo ít nhất một Album để chứa bài hát!")
    }
}




onMounted(() => {
    getBills();
    useActivity.fetchUserData();
    console.log(authStore.user)
})
</script>

<template>
    <div class="text-white space-y-10 rounded-[24px] bg-[#1D1512] w-full h-[76%]">
        <div class="px-10 py-6 h-52 flex items-center bg-gradient-to-b from-[#1D1512]/20 to-[#312825]/100 space-x-7 ">
            <div class="rounded-full bg-gray-600 w-[160px] h-[160px] aspect-square">
                <img class="rounded-full w-[160px] h-[160px] aspect-square object-cover"
                    :src="authStore.user.avatar_path" alt="" @error="event => event.target.src = defaultImgage">
            </div>
            <div class="space-y-2">
                <p class="text-lg text-zinc-400">Hồ sơ</p>
                <h1 class="text-6xl font-bold">{{ authStore.user.name }}</h1>
                <p class="mt-1 text-lg text-zinc-400">{{ myPlaylistList.length }} Playlist • {{ followArtistList.length
                }} nghệ sĩ đang theo dõi</p>
            </div>
        </div>

        <div class="relative px-10 overflow-y-auto scrollbar-style h-[calc(100vh-450px)]">
            <button class="p-2 mb-5 text-3xl rounded-full text-zinc-400 hover:bg-zinc-600 hover:text-white"
                @click="openMenu = !openMenu">
                <Icon icon="ph:dots-three-outline-fill" />
            </button>
            <span v-if="openMenu" class="absolute bg-[#282828] w-[200px] z-20 left-10 top-14 p-1">
                <ul class="text-gray-200 font-semibold text-[14px]">
                    <li class="px-3 py-2 hover:bg-[#3E3D3D] cursor-pointer" @click="openEditProfile = true">Chỉnh sửa hồ
                        sơ</li>
                    <li class="px-3 py-2 hover:bg-[#3E3D3D] cursor-pointer" @click="albumCheck">Đăng tải bài
                        hát</li>
                    <li class="px-3 py-2 hover:bg-[#3E3D3D] cursor-pointer" @click="createAlbum">Tạo một album</li>
                </ul>
            </span>

            <div class="mt-8 mb-3" v-if="myAlbumList.length > 0">
                <h2 class="mb-4 text-lg font-semibold">Album của tôi</h2>
                <div class="overflow-x-auto scrollbar-style">
                    <div class="flex px-1 py-2 space-x-7 w-max">
                        <div v-for="item in myAlbumList" :key="item.id" class="flex-shrink-0 w-48 duration-200 ease-in-out cursor-pointer group hover:scale-105 hover:brightness-105"
                            @click="useView.selectItem(item); useView.setComponent('PlaylistPage'); useView.setPlaylistData(item);">                     
                            <div class="w-full h-48 mb-2 rounded bg-zinc-700">
                                <img class="object-cover w-full h-full rounded-xl" :src="item.thumbnail_path" @error="event => event.target.src = defaultImgage">
                            </div>
                            <div class="flex justify-between">
                                <div>
                                    <p class="text-lg font-medium">{{ item.name }}</p>
                                    <p class="text-sm text-zinc-400">Năm {{ new Date(item.created_at).getFullYear() }}</p>
                                </div>
                                <button class=" hover:bg-white/5 p-2 rounded text-[#FFE5D6]/50 transition-all duration-200"
                                    @click.stop="console.log(item); useModal.setPlaylistEditData(item); openEditAlbum = true">
                                    <Icon icon="material-symbols:edit-square-rounded" class="text-xl " />
                                </button>
                            </div>                 
                        </div>
                    </div>               
                </div>
            </div>

            <div class="mt-8 mb-3" v-if="mySongList.length > 0">
                <h2 class="mb-4 text-lg font-semibold">Danh sách bài hát của tôi</h2>
                <div class="overflow-x-auto scrollbar-style">
                    <div class="flex px-1 py-2 space-x-7 w-max">
                        <div v-for="item in mySongList" :key="item.id" class="relative flex-shrink-0 duration-200 ease-in-out cursor-pointer group hover:scale-105 hover:brightness-105"
                            @click="useSong.playThisSong(item)">
                            <div class="w-32 h-32 mb-5 rounded-full bg-zinc-700">
                                <img class="object-cover w-full h-full rounded-full" :src="item.thumbnail_path" @error="event => event.target.src = defaultImgage">
                            </div>
                            <div class="flex justify-between">
                                <div>
                                    <p class="font-medium ">{{ item.name }}</p>
                                </div>
                                <button @click.stop="useSong.addSongToWaitlist(item);"
                                    class=" hover:bg-white/5 p-1 rounded text-[#FFE5D6]/50 mr-4">
                                    <Icon icon="material-symbols:home-storage-outline" class="text-2xl " />
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>



            <div v-if="myPlaylistList.length > 0">
                <h2 class="mb-4 text-lg font-semibold">Playlist của tôi</h2>
                <div class="overflow-x-auto scrollbar-style">
                    <div class="flex px-1 py-2 space-x-7 w-max">
                        <div v-for="item in myPlaylistList" :key="item.id" class="flex-shrink-0 w-32 duration-200 ease-in-out cursor-pointer group hover:scale-105 hover:brightness-105"
                            @click="useView.selectItem(item); useView.setComponent('PlaylistPage'); useView.setPlaylistData(item);">
                            <div class="w-full h-32 mb-2 rounded bg-zinc-700">
                                <img class="object-cover w-full h-full rounded-xl" :src="item.thumbnail_path"
                                    @error="event => event.target.src = defaultImgage">
                            </div>
                            <p class="font-medium">{{ item.name }}</p>
                        </div>
                    </div>              
                </div>
            </div>

            <div class="mt-8" v-if="followArtistList.length > 0">
                <h2 class="mb-4 text-lg font-semibold">Đang theo dõi</h2>
                <div class="overflow-x-auto scrollbar-style">
                    <div class="flex px-1 py-2 space-x-7 w-max">
                        <div v-for="item in followArtistList" :key="item.id" class="flex-shrink-0 duration-200 ease-in-out cursor-pointer group hover:scale-105 hover:brightness-105"
                            @click="useView.selectItem(item.artist); useView.setComponent('ArtistPage'); useView.setArtistData(item.artist);">
                            <div class="w-32 h-32 mb-5 rounded-full bg-zinc-700">
                                <img class="object-cover w-full h-full rounded-full" :src="item.artist.avatar_path" @error="event => event.target.src = defaultImgage">
                            </div>
                            <p class="text-sm text-zinc-400">{{ item.artist.name }}</p>
                        </div>
                    </div>
                </div>
            </div>


            <div v-if="bills.length > 0">
                <h2 class="mt-8 mb-4 text-lg font-semibold">Lịch sử thanh toán</h2>
                <div class="max-h-[30rem] overflow-y-auto border border-gray-700 rounded-lg scrollbar-style">               
                    <table class="min-w-full bg-[#1a1a1a]">
                        <thead class="sticky top-0 bg-[#2a2a2a]">
                            <tr>
                                <th class="px-4 py-3 text-sm font-semibold text-left text-[#FFE5D6]">Mã giao dịch</th>
                                <th class="px-4 py-3 text-sm font-semibold text-left text-[#FFE5D6]">Ngày thanh toán</th>
                                <th class="px-4 py-3 text-sm font-semibold text-left text-[#FFE5D6]">Số tiền</th>
                                <th class="px-4 py-3 text-sm font-semibold text-left text-[#FFE5D6]">Tên</th>
                                <th class="px-4 py-3 text-sm font-semibold text-left text-[#FFE5D6]">Gói dịch vụ</th>
                                <th class="px-4 py-3 text-sm font-semibold text-left text-[#FFE5D6]">Trạng thái</th>
                            </tr>
                        </thead>

                        <tbody>
                            <tr
                                v-for="item in bills"
                                :key="item.id"
                                class="border-t border-gray-700 hover:bg-[#2a2a2a] transition-colors"
                            >
                                <td class="px-4 py-3 text-[#B0B0B0]">{{ item.id }}</td>
                                <td class="px-4 py-3 text-[#B0B0B0]">{{  new Date(item.created_at).toLocaleDateString('vi-VN') }}</td>
                                <td class="px-4 py-3 text-[#B0B0B0]">{{ item.playlist_id ? item.playlist.price : item.song.price }} vnd</td>
                                <td class="px-4 py-3 text-[#B0B0B0]">{{ item.playlist_id ? item.playlist.name : item.song.name }}</td>
                                <td class="px-4 py-3 text-[#B0B0B0]">{{ item.playlist_id ? 'Album' : 'Bài hát' }}</td>
                                <td
                                    class="px-4 py-3 font-medium"
                                    :class="{
                                    'text-green-500': item.status == 2,
                                    'text-yellow-500': item.status == 1,
                                    }"
                                >
                                {{ item.status == 1 ? 'Thanh toán thất bại' : 'Thanh toán thành công' }}
                                </td>
                            </tr>
                        </tbody>
                    </table>
                    
                </div>
            </div>




        </div>
    </div>
</template>