<script setup lang="ts">
import { nextTick, onMounted, ref } from "vue";
import Chat from "./components/chat.vue";
import {
    addDoc,
    collection,
    deleteDoc,
    doc,
    getDocs,
    orderBy,
    query,
} from "firebase/firestore";
import { db } from "./firebase/firebase";
import type { ChatProps } from "./types/chatProps";
import { GoogleGenAI } from "@google/genai";

const chat = ref<string>("");
const messages = ref<ChatProps[]>([]);
const loading = ref<boolean>(false);
const bottomRef = ref<HTMLElement | null>(null);
const messagesContainer = ref<HTMLElement | null>(null);
const isAtBottom = ref<boolean>(true);
const pastMessages = localStorage.getItem("messages")
    ? JSON.parse(localStorage.getItem("messages") || "[]")
    : [];
const apiKey = import.meta.env.VITE_GEMINI_API_KEY;
const ai = new GoogleGenAI({ apiKey: apiKey });

const scrollToBottom = async () => {
    await nextTick();
    bottomRef.value?.scrollIntoView({ behavior: "smooth" });
    isAtBottom.value = true;
};

const handleScroll = () => {
    if (messagesContainer.value) {
        const { scrollTop, scrollHeight, clientHeight } =
            messagesContainer.value;
        isAtBottom.value = scrollHeight - scrollTop - clientHeight < 100;
    }
};

const deleteMessages = async () => {
    try {
        localStorage.removeItem("messages");
        messages.value = [];
        const querySnapshot = await getDocs(collection(db, "messages"));
        const deletePromises = querySnapshot.docs.map((document) =>
            deleteDoc(doc(db, "messages", document.id)),
        );
        await Promise.all(deletePromises);
        console.log("모든 메시지가 삭제되었습니다.");
    } catch (error) {
        console.error("삭제 중 오류 발생:", error);
    }
};

const formatDate = (timestamp: number): string => {
    const date = new Date(timestamp);
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, "0");
    const day = String(date.getDate()).padStart(2, "0");
    return `${year}년 ${month}월 ${day}일`;
};

const groupedMessages = () => {
    const groups: { date: string; messages: ChatProps[] }[] = [];
    let currentDate = "";

    messages.value.forEach((msg) => {
        const dateStr = formatDate(msg.timestamp);
        if (dateStr !== currentDate) {
            currentDate = dateStr;
            groups.push({ date: dateStr, messages: [msg] });
        } else {
            groups[groups.length - 1]?.messages.push(msg);
        }
    });

    return groups;
};

const handleSendMessage = async () => {
    if (chat.value.trim() === "" || loading.value) return;
    try {
        loading.value = true;
        const userMessage = chat.value;
        const userDocRef = await addDoc(collection(db, "messages"), {
            msg: userMessage,
            timestamp: Date.now(),
            isAi: false,
        });
        chat.value = "";
        messages.value.push({
            chatId: userDocRef.id,
            msg: userMessage,
            timestamp: Date.now(),
            isAi: false,
        });
        await scrollToBottom();

        const systemPrompt = `당신은 사용자의 감정 일기를 읽고 음악적 영감을 주는 '음악 큐레이터'입니다.

[응답 가이드라인]
- 감정 분석: 사용자의 글에서 핵심 감정을 파악해 공감해줍니다.
- 코드 진행 추천: 해당 감정에 어울리는 4마디 코드 진행을 제안합니다. (예: Cmaj7 - G - Am - F)
- 분위기 설명: 추천하는 악기, 템포(BPM), 전반적인 무드를 설명합니다.

반드시 음악 전문가의 시선에서 답변하되, 따뜻하고 친절한 말투를 유지하세요.

    [대화 가이드라인]
    - 만약 읽기 어려운 글이 있다면, "글이 조금 어려운데, 혹시 이런 감정이 담긴 건가요?"라며 친절하게 재확인해 주세요.
    - 사용자가 감정을 명확히 표현하지 않는다면, 단호하게 "감정을 조금 더 자세히 설명해 주실 수 있나요?"라고 물어보세요.
`;

        const response = await ai.models.generateContent({
            model: "gemini-1.5-flash",
            contents: `${systemPrompt}\n\n사용자: ${userMessage}`,
        });

        const aiResponseText =
            response.text?.toString().replace(/\*/g, "").replace("#", "") || "";

        const aiDocRef = await addDoc(collection(db, "messages"), {
            msg: aiResponseText,
            timestamp: Date.now(),
            isAi: true,
        });

        messages.value.push({
            chatId: aiDocRef.id,
            msg: aiResponseText,
            timestamp: Date.now(),
            isAi: true,
        });

        console.log("메시지 전송 완료");
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
        const q = query(
            collection(db, "messages"),
            orderBy("timestamp", "asc"),
        );
        const querySnapshot = await getDocs(q);
        messages.value = [];
        querySnapshot.forEach((doc) => {
            messages.value.push({
                chatId: doc.id,
                msg: doc.data().msg,
                timestamp: doc.data().timestamp,
                isAi: doc.data().isAi || false,
            });
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
            <h2
                class="font-semibold cursor-pointer"
                v-on:click="deleteMessages"
            >
                새로운 채팅
            </h2>
        </div>
        <!-- 여기가 밑으로 가는 거 보여주는 곳-->

        <!-- 여기서 채팅 넣는 곳 -->
        <div
            ref="messagesContainer"
            class="flex flex-col gap-10 scroll-auto"
            @scroll="handleScroll"
        >
            <p v-for="el in pastMessages && loading" :key="el.chatId">
                <Chat
                    :chatId="el.chatId"
                    :msg="el.msg"
                    :timestamp="el.timestamp"
                    :isAi="el.isAi"
                />
            </p>
            <h3
                v-if="messages.length === 0"
                class="text-gray-500 text-center font-semibold"
            >
                <h2 class="text-xl font-bold">좋은 하루예요!</h2>
                <br />
                안녕하세요! 음악 아이디어를 공유해보세요.
                <br />AI가 함께 대화하며 아이디어를 발전시켜 드립니다.
            </h3>
            <!-- 날짜별 메시지 그룹화 -->
            <template v-for="group in groupedMessages()" :key="group.date">
                <!-- 날짜 구분선 -->
                <div class="flex items-center gap-3 my-4">
                    <div class="flex-1 border-t border-gray-300"></div>
                    <span class="text-xs text-gray-500 px-2">{{
                        group.date
                    }}</span>
                    <div class="flex-1 border-t border-gray-300"></div>
                </div>
                <!-- 해당 날짜의 메시지들 -->
                <div v-for="el in group.messages" :key="el.chatId">
                    <Chat
                        :chatId="el.chatId"
                        :msg="el.msg"
                        :timestamp="el.timestamp"
                        :isAi="el.isAi"
                    />
                </div>
            </template>
            <div v-if="loading">AI가 답변을 생성 중입니다...</div>
            <div ref="bottomRef"></div>
        </div>
        <!-- 여기가 밑으로 가는 거 보여주는 곳-->
        <!-- 메시지 입력창  -->
        <div class="w-full sticky bottom-20 left-0 px-4">
            <div
                v-if="!isAtBottom"
                class="w-full flex justify-center mb-4 cursor-pointer"
            >
                <img
                    src="/arrow.svg"
                    alt="맨 아래로 이동"
                    class="w-8 h-8 hover:opacity-80 transition-opacity"
                    @click="scrollToBottom"
                />
            </div>
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
                :src="chat.trim() ? '/chatarrow.svg' : '/opchatarrow.svg'"
                class="absolute right-8 bottom-6 cursor-pointer hover:opacity-80 transition-opacity duration-600"
                alt="메시지 전송"
                @click="handleSendMessage"
            />
        </div>
    </div>
</template>
