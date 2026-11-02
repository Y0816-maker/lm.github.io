<template>
  <div class="simulator-container">
    <h1 class="title">OSI 七层模型：封装与解封装模拟器</h1>
    
    <!-- 输入区域 -->
    <div class="input-area">
      <input 
        v-model="userMessage" 
        placeholder="输入你要发送的消息..." 
        class="message-input"
      />
      <button @click="encapsulateData" class="send-btn">📦 开始封装</button>
      <button @click="decapsulateData" class="recv-btn" :disabled="layers.length === 0">📬 开始解封装</button>
    </div>

    <!-- 层级展示区域 -->
    <div class="layers-wrapper">
      <transition-group name="layer-fade">
        <div 
          v-for="(layer, index) in layers" 
          :key="layer.name" 
          class="layer-card"
          :class="{ 'unwrap-card': layer.isUnwrap }"
          :style="{ animationDelay: `${index * 0.1}s` }"
        >
          <div class="layer-header">
            <span class="layer-index" :class="{ 'unwrap-index': layer.isUnwrap }">
              {{ layer.isUnwrap ? '📤' : '📥' }} L{{ layer.level }}
            </span>
            <span class="layer-name">{{ layer.name }}</span>
          </div>
          <div class="layer-result">{{ layer.result }}</div>
        </div>
      </transition-group>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const userMessage = ref('');
const layers = ref([]);

// 封装过程 (自上而下)
const encapsulateData = () => {
  if (!userMessage.value.trim()) return;
  layers.value = []; 

  const appData = JSON.stringify({ protocol: 'HTTP', content: userMessage.value });
  layers.value.push({ level: 7, name: '应用层 (Application)', result: appData, isUnwrap: false });

  const presData = btoa(appData); // Base64加密
  layers.value.push({ level: 6, name: '表示层 (Presentation)', result: `Base64加密: ${presData}`, isUnwrap: false });

  const sessData = `SESSION_ID_888|${presData}`;
  layers.value.push({ level: 5, name: '会话层 (Session)', result: sessData, isUnwrap: false });

  const transData = `TCP[8080->80]${sessData}`;
  layers.value.push({ level: 4, name: '传输层 (Transport)', result: transData, isUnwrap: false });

  const netData = `IP[192.168.1.10 to 10.0.0.5]${transData}`;
  layers.value.push({ level: 3, name: '网络层 (Network)', result: netData, isUnwrap: false });

  const linkData = `MAC[AA:BB >> 11:22]${netData}|FCS_OK`;
  layers.value.push({ level: 2, name: '数据链路层 (Data Link)', result: linkData, isUnwrap: false });

  const bitStream = Array.from(linkData)
    .map(c => c.charCodeAt(0).toString(2).padStart(8, '0'))
    .join(' ');
  layers.value.push({ level: 1, name: '物理层 (Physical)', result: bitStream.substring(0, 80) + '...', isUnwrap: false });
};

// 解封装过程 (自下而上)
const decapsulateData = () => {
  if (layers.value.length === 0) return;
  
  // 获取封装后的原始数据
  const rawLinkData = `MAC[AA:BB >> 11:22]IP[192.168.1.10 to 10.0.0.5]TCP[8080->80]SESSION_ID_888|${btoa(JSON.stringify({ protocol: 'HTTP', content: userMessage.value }))}|FCS_OK`;
  
  const unwrapLayers = [];
  let currentData = rawLinkData;

  // 1. 物理层 -> 数据链路层
  unwrapLayers.push({ level: 1, name: '物理层 (Physical)', result: '接收比特流，还原为数据帧...', isUnwrap: true });
  
  const macHeader = 'MAC[AA:BB >> 11:22]';
  const fcsTrailer = '|FCS_OK';
  currentData = currentData.substring(macHeader.length, currentData.length - fcsTrailer.length);
  unwrapLayers.push({ level: 2, name: '数据链路层 (Data Link)', result: `剥离MAC头尾, FCS校验通过: ${currentData.substring(0, 40)}...`, isUnwrap: true });

  // 2. 网络层
  const ipHeader = 'IP[192.168.1.10 to 10.0.0.5]';
  currentData = currentData.substring(ipHeader.length);
  unwrapLayers.push({ level: 3, name: '网络层 (Network)', result: `剥离IP头部, 目标IP匹配: ${currentData.substring(0, 40)}...`, isUnwrap: true });

  // 3. 传输层
  const tcpHeader = 'TCP[8080->80]';
  currentData = currentData.substring(tcpHeader.length);
  unwrapLayers.push({ level: 4, name: '传输层 (Transport)', result: `剥离TCP头部, 端口80接收数据: ${currentData.substring(0, 40)}...`, isUnwrap: true });

  // 4. 会话层
  const sessHeader = 'SESSION_ID_888|';
  currentData = currentData.substring(sessHeader.length);
  unwrapLayers.push({ level: 5, name: '会话层 (Session)', result: `剥离会话控制头: ${currentData.substring(0, 40)}...`, isUnwrap: true });

  // 5. 表示层 (解密)
  const decryptedData = atob(currentData);
  unwrapLayers.push({ level: 6, name: '表示层 (Presentation)', result: `Base64解密完成: ${decryptedData.substring(0, 40)}...`, isUnwrap: true });

  // 6. 应用层
  const appObj = JSON.parse(decryptedData);
  unwrapLayers.push({ level: 7, name: '应用层 (Application)', result: `🎉 成功还原原始消息: "${appObj.content}"`, isUnwrap:true});
  layers.value = unwrapLayers;
};
</script>