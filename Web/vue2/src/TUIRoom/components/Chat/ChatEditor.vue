<template>
  <div
    :class="['chat-editor', chatStore.isMuteChatByMater ? 'disable-editor': '']"
  >
    <textarea
      ref="editorInputEle"
      v-model="sendMsg"
      class="content-bottom-input"
      :disabled="chatStore.isMuteChatByMater"
      :placeholder="chatStore.isMuteChatByMater ? '已被主持人禁言' : '说点什么'"
      @keyup.enter="sendMessage"
    />
    <div v-if="!chatStore.isMuteChatByMater" class="chat-editor-toolbar">
      <div class="left-section">
        <emoji @choose-emoji="handleChooseEmoji"></emoji>
      </div>
      <div :class="['send-btn', `${sendMsg.length > 0 ? 'active' : ''}`]" @click="sendMessage">发送</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import TUIRoomCore from '../../tui-room-core';
import { useChatStore } from '../../stores/chat';
import { Message } from 'element-ui';
import emoji from './EditorTools/emoji.vue';

const editorInputEle = ref();

const chatStore = useChatStore();
const sendMsg = ref('');
const sendMessage = async () => {
  const msg = sendMsg.value.replace('\n', '');
  sendMsg.value = '';
  if (msg === '') {
    return;
  }
  try {
    const tuiResponse = await TUIRoomCore.sendChatMessage(msg);
    const { code, data: message } = tuiResponse;
    if (code === 0) {
      // 消息发送成功
      chatStore.updateMessageList(message);
    }
  } catch (e) {
    // 消息发送失败
    Message.error('发送消息失败');
  }
};

const handleChooseEmoji = (emojiName: string) => {
  sendMsg.value += emojiName;
  editorInputEle.value.focus();
};

</script>

<style lang="scss" scoped>
@import '../../assets/style/var.scss';

  .chat-editor {
    height: 188px;
    background: #2E323D;
    box-sizing: border-box;
    &.disable-editor {
      textarea {
      }
    }
    textarea {
      height: 138px;
      color: #ffff;
      width: 100%;
      background: #2E323D;
      border: none;
      box-sizing: border-box;
      padding: 12px 14px;
      caret-color: #ffffff;
      resize: none;
      &:focus-visible {
        outline: none;
      }
    }
    .send-btn {
      padding: 6px 18px;
      background: #3D4352;
      border-radius: 2px;
      font-size: 14px;
      color: #CFD4E6;
      &:hover {
        cursor: pointer;
        background: $primaryHighLightColor;
        color: $whiteColor;
      }
      &.active {
        background: $primaryHighLightColor;
        color: $whiteColor;
      }
    }
    .chat-editor-toolbar {
      height: 44px;
      padding: 0 14px 12px;
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: space-between;
    }
  }
</style>
