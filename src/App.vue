<script setup lang="ts">
import { ref } from "vue";
import Chat from "./components/chat.vue";
const chat = ref<string>("");

const handleSendMessage = () => {
    if (chat.value.trim()) {
        console.log("메시지 전송:", chat.value);
        chat.value = "";
    }
};

const handleKeydown = (event: KeyboardEvent) => {
    if (event.key === "Enter" && !event.shiftKey) {
        event.preventDefault();
        handleSendMessage();
    }
};
</script>

<template>
    <div class="flex flex-col gap-10 pb-30 relative">
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
            <Chat
                chatId="1"
                msg="Hello, world!adpgojapodgjpoadjgpoajdgpojadpogjadpogjpaodgjpoadjgpodajogjdapogjpoadjgpoadjgpoadjpgojdapojgapojdapogjadpogjpoadgjpodj"
                :timestamp="Date.now()"
                :isAi="true"
            />
            <Chat
                chatId="1"
                msg="Hello, world!adpgojapodgjpoadjgpoajdgpojadpogjadpogjpaodgjpoadjgpodajogjdapogjpoadjgpoadjgpoadjpgojdapojgapojdapogjadpogjpoadgjpodj"
                :timestamp="Date.now()"
                :isAi="false"
            />
            <Chat
                chatId="1"
                msg="Hello, world!adpgojapodgjpoadjgpoajdgpojadpogjadpogjpaodgjpoadjgpodajogjdapogjpoadjgpoadjgpoadjpgojdapojgapojdapogjadpogjpoadgjpodj"
                :timestamp="Date.now()"
                :isAi="true"
            />
            <Chat
                chatId="1"
                msg="Hello, world!adpgojapodgjpoadjgpoajdgpojadpogjadpogjpaodgjpoadjgpodajogjdapogjpoadjgpoadjgpoadjpgojdapojgapojdapogjadpogjpoadgjpodj"
                :timestamp="Date.now()"
                :isAi="true"
            />
            <Chat
                chatId="1"
                msg="Hello, world!adpgojapodgjpoadjgpoajdgpojadpogjadpogjpaodgjpoadjgpodajogjdapogjpoadjgpoadjgpoadjpgojdapojgapojdapogjadpogjpoadgjpodj"
                :timestamp="Date.now()"
                :isAi="true"
            />
            <Chat
                chatId="1"
                msg="Hello, world!adpgojapodgjpoadjgpoajdgpojadpogjadpogjpaodgjpoadjgpodajogjdapogjpoadjgpoadjgpoadjpgojdapojgapojdapogjadpogjpoadgjpodj"
                :timestamp="Date.now()"
                :isAi="true"
            />
        </div>
        <!-- 메시지 입력창  -->
        <form class="w-full h-20 sticky bottom-20 left-0">
            <textarea
                height="100%"
                v-model="chat"
                placeholder="메시지를 입력하세요..."
                class="max-w-full min-w-full bg-white rounded-md p-4 border-none outline-none break-all whitespace-pre-wrap resize-none"
                @keydown="handleKeydown"
                :style="{
                    boxShadow: '0px 4px 4px rgba(233, 233, 233, 0.25)',
                    minHeight: '60px',
                }"
            />
            <img
                src="/public/chatarrow.svg"
                class="absolute right-5 bottom-5 cursor-pointer"
                alt="이미지가 안보임 사망함."
            />
        </form>
    </div>
</template>
