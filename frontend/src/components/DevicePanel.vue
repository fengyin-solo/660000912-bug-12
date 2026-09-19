<template>
  <div style="width:300px;padding:16px;overflow:auto;border-left:1px solid #e0e0e0;height:100vh;box-sizing:border-box">
    <h3 style="margin:0 0 12px;display:flex;align-items:center;justify-content:space-between">
      <span>📡 设备列表</span>
      <button @click="emit('add-device')"
        style="padding:6px 12px;background:#1b5e20;color:#fff;border:none;border-radius:6px;cursor:pointer;font-size:12px;font-weight:500">
        ➕ 注册
      </button>
    </h3>
    <div style="display:flex;gap:8px;margin-bottom:16px">
      <span style="font-size:12px;padding:2px 8px;border-radius:12px;background:#e8f5e9">🟢 {{ store.onlineCount }} 在线</span>
      <span style="font-size:12px;padding:2px 8px;border-radius:12px;background:#ffebee">⚠️ {{ store.alertCount }} 告警</span>
    </div>
    <div v-for="d in store.devices" :key="d.id"
      @click="handleDeviceClick(d.id)"
      @mouseenter="handleHover(d.id)"
      @mouseleave="handleHover(null)"
      :style="{ display:'flex', alignItems:'center', gap:'10px', padding:'10px', marginBottom:'8px',
        borderRadius:'8px', border:'2px solid ' + (isSelected(d.id) ? '#1976d2' : isHovered(d.id) ? '#90caf9' : (d.status === 'alert' ? '#ffcc80' : '#e0e0e0')),
        background: isSelected(d.id) ? '#e3f2fd' : isHovered(d.id) ? '#f0f7ff' : (d.status === 'alert' ? '#fff3e0' : '#fff'),
        cursor:'pointer', transition:'all 0.2s ease' }">
      <span :style="{ width:'10px', height:'10px', borderRadius:'50%',
        background: d.status === 'online' ? '#4caf50' : d.status === 'alert' ? '#ff9800' : '#9e9e9e',
        boxShadow: isSelected(d.id) ? '0 0 0 3px rgba(25,118,210,0.3)' : 'none' }"></span>
      <div style="flex:1">
        <div :style="{ fontWeight: isSelected(d.id) ? 700 : 500, fontSize:'13px', color:'#333', display:'flex', alignItems:'center', gap:'6px' }">
          {{ d.name }}
          <span v-if="d.groupId && getGroup(d.groupId)"
            :style="{ fontSize:'10px', padding:'1px 6px', borderRadius:'8px', background: getGroup(d.groupId)!.color + '20', color: getGroup(d.groupId)!.color }">
            {{ getGroup(d.groupId)!.name }}
          </span>
        </div>
        <div style="font-size:11px;color:#888">🔋 {{ d.battery }}% · 🌡 {{ d.temperature }}°C</div>
      </div>
      <span v-if="d.status === 'alert'" style="font-size:10px;color:#ff9800">⚠️</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onUnmounted } from 'vue';
import { useIotStore } from '../stores/iot';
const store = useIotStore();

const emit = defineEmits<{
  (e: 'add-device'): void;
}>();

function getGroup(groupId: string) {
  return store.getGroupById(groupId);
}

function isSelected(id: string) {
  return store.highlightedDeviceId === id;
}

function isHovered(id: string) {
  return store.hoveredDeviceId === id;
}

// 点击 = 明确选中，持久保持，驱动地图定位与详情
function handleDeviceClick(id: string) {
  store.setHighlightedDevice(id);
}

// 悬停 = 临时状态，只写 hoveredDeviceId，不触碰选中状态
function handleHover(id: string | null) {
  store.setHoveredDevice(id);
}

// 面板卸载时清理可能残留的悬停状态（DOM 移除不会触发 mouseleave）
onUnmounted(() => {
  store.setHoveredDevice(null);
});
</script>
