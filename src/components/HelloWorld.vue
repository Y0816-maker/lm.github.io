<template>
  <div class="osi-simulator">
    <h2>OSI 七层模型可视化模拟器</h2>
    <input v-model="userMessage" placeholder="输入你要发送的消息..." />
    <button @click="encapsulateData">开始封装</button>
    
    <div class="layers-container">
      <div v-for="layer in layers" :key="layer.name" class="layer-card">
        <h3>{{ layer.name }}</h3>
        <p>处理结果: <code>{{ layer.result }}</code></p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const userMessage = ref('');
const layers = ref([]);

// 模拟 Base64 加密（表示层）
const simpleEncrypt = (str) => btoa(str); 

const encapsulateData = () => {
  if (!userMessage.value) return;
  layers.value = []; // 清空上一次的记录

  // 1. 应用层
  const appData = JSON.stringify({ protocol: 'HTTP', content: userMessage.value });
  layers.value.push({ name: '1. 应用层 (Application)', result: appData });

  // 2. 表示层
  const presData = simpleEncrypt(appData);
  layers.value.push({ name: '2. 表示层 (Presentation)', result: presData });

  // 3. 会话层
  const sessData = `SESSION_ID_888|${presData}`;
  layers.value.push({ name: '3. 会话层 (Session)', result: sessData });

  // 4. 传输层
  const transData = `TCP[8080->80]${sessData}`;
  layers.value.push({ name: '4. 传输层 (Transport)', result: transData });

  // 5. 网络层
  const netData = `IP[192.168.1.10 to 10.0.0.5]${transData}`;
  layers.value.push({ name: '5. 网络层 (Network)', result: netData });

  // 6. 数据链路层
  const linkData = `MAC[AA:BB >> 11:22]${netData}|FCS_OK`;
  layers.value.push({ name: '6. 数据链路层 (Data Link)', result: linkData });

  // 7. 物理层
  // 将字符串转为简单的二进制展示
  const bitStream = Array.from(linkData).map(c => c.charCodeAt(0).toString(2).padStart(8, '0')).join(' ');
  layers.value.push({ name: '7. 物理层 (Physical)', result: bitStream.substring(0, 50) + '...' });
};
</script>
