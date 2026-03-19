<script setup lang="ts">
import { nextTick, onMounted, ref } from "vue";
import Chat from "./components/chat.vue";
import { addDoc, collection, getDocs } from "firebase/firestore";
import { db } from "./firebase/firebase";
import type { ChatProps } from "./types/chatProps";

const chat = ref<string>("");
const messages = ref<ChatProps[]>([]);
const loading = ref<boolean>(false);
const bottomRef = ref<HTMLElement | null>(null);
const pastMessages = localStorage.getItem("messages")
    ? JSON.parse(localStorage.getItem("messages") || "[]")
    : [];

const scrollToBottom = async () => {
    await nextTick();
    bottomRef.value?.scrollIntoView({ behavior: "smooth" });
};

const handleSendMessage = async () => {
    if (chat.value.trim() === "" || loading.value) return;
    try {
        loading.value = true;
        const docRef = await addDoc(collection(db, "messages"), {
            msg: chat.value,
            timestamp: Date.now(),
            isAi: false,
        });
        messages.value.push({
            chatId: docRef.id,
            msg: chat.value,
            timestamp: Date.now(),
            isAi: false,
        });
        console.log("메시지 전송:", chat.value);
        chat.value = "";
    } catch (error) {
        console.error("메시지 전송 실패:", error);
    } finally {
        await scrollToBottom();
        loading.value = false;
    }
};
onMounted(async () => {
    try {
        loading.value = true;
        const querySnapshot = await getDocs(collection(db, "messages"));
        messages.value = [];
        querySnapshot.forEach((doc) => {
            messages.value.push({
                chatId: doc.id,
                msg: doc.data().msg,
                timestamp: doc.data().timestamp,
                isAi: doc.data().isAi || false,
            });
            console.log(doc.id, "=>", doc.data());
            localStorage.setItem("messages", JSON.stringify(messages.value));
        });
    } catch (error) {
        console.error("메시지 로드 실패:", error);
    } finally {
        await scrollToBottom();
        loading.value = false;
    }
});
</script>

<template>
    <div class="flex flex-col gap-10 pb-10 relative">
        <div
            class="w-full flex justify-between items-center sticky p-4 top-0 z-10 bg-[#f2f5f7]"
        >
            <img
                src="/public/logo.svg"
                class="w-20"
                alt="이미지가 안보임 사망함."
            />
            <h2 class="font-semibold">새로운 채팅</h2>
        </div>
        <!-- 여기서 채팅 넣는 곳 -->
        <div class="flex flex-col gap-10 scroll-auto">
            <p
                v-if="loading"
                class="text-gray-500 flex w-full justify-center"
            ></p>
            <p v-for="el in pastMessages && loading" :key="el.chatId">
                <Chat
                    :chatId="el.chatId"
                    :msg="el.msg"
                    :timestamp="el.timestamp"
                    :isAi="el.isAi"
                />
            </p>
            <h3
                v-if="messages.length === 0 && !loading"
                class="text-gray-500 text-center font-semibold"
            >
                안녕하세요! 음악 아이디어를 공유해보세요.
                <br />AI가 함께 대화하며 아이디어를 발전시켜 드립니다.
            </h3>
            <div v-for="el in messages" :key="el.chatId">
                <Chat
                    :chatId="el.chatId"
                    :msg="el.msg"
                    :timestamp="el.timestamp"
                    :isAi="el.isAi"
                />
            </div>
            <div ref="bottomRef"></div>
        </div>
        <!-- 메시지 입력창  -->
        <div class="w-full sticky bottom-20 left-0 px-4">
            <textarea
                v-model="chat"
                placeholder="메시지를 입력하세요..."
                class="w-full bg-white rounded-md p-4 border-none outline-none break-all whitespace-pre-wrap resize-none"
                @keydown.enter.prevent="handleSendMessage"
                :style="{
                    boxShadow: '0px 4px 4px rgba(233, 233, 233, 0.25)',
                    minHeight: '60px',
                }"
            />
            <img
                src="/public/chatarrow.svg"
                class="absolute right-8 bottom-6 cursor-pointer hover:opacity-80"
                alt="메시지 전송"
                @click="handleSendMessage"
            />
        </div>
    </div>
</template>
