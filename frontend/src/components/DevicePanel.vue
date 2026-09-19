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

    <!-- 明确选中设备的详情：直接派生自设备列表，列表删除/刷新后自动同步，不会指向旧目标 -->
    <div v-if="selectedDevice"
      style="margin-bottom:16px;padding:12px;border-radius:8px;border:2px solid #1976d2;background:#f5faff">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:8px">
        <div style="display:flex;align-items:center;gap:6px;font-weight:700;font-size:14px;color:#1565c0">
          <span :style="{ width:'10px', height:'10px', borderRadius:'50%',
            background: selectedDevice.status === 'online' ? '#4caf50' : selectedDevice.status === 'alert' ? '#ff9800' : '#9e9e9e' }"></span>
          {{ selectedDevice.name }}
        </div>
        <button @click="store.selectDevice(null)" title="取消选择"
          style="background:none;border:none;cursor:pointer;color:#90a4ae;font-size:14px;line-height:1;padding:2px">✕</button>
      </div>
      <div v-if="selectedDevice.groupId && getGroup(selectedDevice.groupId)" style="margin-bottom:8px">
        <span :style="{ fontSize:'11px', padding:'2px 8px', borderRadius:'8px',
          background: getGroup(selectedDevice.groupId)!.color + '20', color: getGroup(selectedDevice.groupId)!.color }">
          🏷 {{ getGroup(selectedDevice.groupId)!.name }}
        </span>
        <span :style="{ fontSize:'11px', padding:'2px 8px', borderRadius:'8px', marginLeft:'6px',
          background: statusMeta.bg, color: statusMeta.color }">
          {{ statusMeta.text }}
        </span>
      </div>
      <div style="font-size:12px;color:#555;line-height:1.9">
        <div>🔋 电量：<b>{{ selectedDevice.battery }}%</b></div>
        <div>🌡 温度：<b>{{ selectedDevice.temperature }}°C</b></div>
        <div>📍 位置：{{ selectedDevice.lat.toFixed(4) }}, {{ selectedDevice.lng.toFixed(4) }}</div>
        <div>🕐 最近上报：{{ formatLastSeen(selectedDevice.lastSeen) }}</div>
        <div>🔔 未处理告警：<b>{{ store.getDeviceAlertsCount(selectedDevice.id) }}</b> 条</div>
      </div>
      <div style="display:flex;gap:6px;margin-top:10px">
        <button @click="handleLocate"
          style="flex:1;padding:6px 8px;background:#1976d2;color:#fff;border:none;border-radius:6px;cursor:pointer;font-size:12px">
          🗺 定位
        </button>
        <button @click="handleDelete"
          style="flex:1;padding:6px 8px;background:#fff;color:#c62828;border:1px solid #c62828;border-radius:6px;cursor:pointer;font-size:12px">
          🗑 删除
        </button>
      </div>
    </div>

    <div v-for="d in store.devices" :key="d.id"
      @click="handleDeviceClick(d.id)"
      @mouseenter="handleHover(d.id)"
      @mouseleave="handleHover(null)"
      :style="{ display:'flex', alignItems:'center', gap:'10px', padding:'10px', marginBottom:'8px',
        borderRadius:'8px', border:'2px solid ' + rowBorderColor(d),
        background: rowBgColor(d),
        cursor:'pointer', transition:'all 0.2s ease' }">
      <span :style="{ width:'10px', height:'10px', borderRadius:'50%',
        background: d.status === 'online' ? '#4caf50' : d.status === 'alert' ? '#ff9800' : '#9e9e9e',
        boxShadow: store.activeHighlightDeviceId === d.id ? '0 0 0 3px rgba(25,118,210,0.3)' : 'none' }"></span>
      <div style="flex:1">
        <div style="font-weight: store.activeHighlightDeviceId === d.id ? 700 : 500;font-size:13px;color:#333;display:flex;align-items:center;gap:6px">
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
import { computed } from 'vue';
import { useIotStore } from '../stores/iot';
import type { Device } from '../types';
const store = useIotStore();

const emit = defineEmits<{
  (e: 'add-device'): void;
}>();

// 详情只认明确选中且仍存在于列表中的设备
const selectedDevice = computed<Device | null>(() => store.selectedDevice);

const statusMeta = computed(() => {
  switch (selectedDevice.value?.status) {
    case 'online':
      return { text: '在线', color: '#2e7d32', bg: '#e8f5e9' };
    case 'alert':
      return { text: '告警', color: '#e65100', bg: '#fff3e0' };
    case 'offline':
      return { text: '离线', color: '#616161', bg: '#f5f5f5' };
    default:
      return { text: '未知', color: '#616161', bg: '#f5f5f5' };
  }
});

function getGroup(groupId: string) {
  return store.getGroupById(groupId);
}

function formatLastSeen(iso: string): string {
  const diff = Date.now() - new Date(iso).getTime();
  if (Number.isNaN(diff)) return '—';
  if (diff < 60000) return '刚刚';
  if (diff < 3600000) return Math.floor(diff / 60000) + ' 分钟前';
  if (diff < 86400000) return Math.floor(diff / 3600000) + ' 小时前';
  return new Date(iso).toLocaleString('zh-CN', { month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit' });
}

function rowBorderColor(d: Device) {
  if (store.activeHighlightDeviceId === d.id) return '#1976d2';
  return d.status === 'alert' ? '#ffcc80' : '#e0e0e0';
}

function rowBgColor(d: Device) {
  if (store.hoveredDeviceId === d.id) return '#e3f2fd';
  if (store.selectedDeviceId === d.id) return '#e3f2fd';
  return d.status === 'alert' ? '#fff3e0' : '#fff';
}

// 明确选中：与临时悬停分离，点击后列表、地图、详情始终以它为准
function handleDeviceClick(id: string) {
  store.selectDevice(id);
}

// 临时悬停：进入高亮、离开恢复，绝不改动明确选中
function handleHover(id: string | null) {
  store.setHoveredDevice(id);
}

// 重新定位到当前选中设备（重复点击也生效）
function handleLocate() {
  store.locateSelectedDevice();
}

function handleDelete() {
  const device = selectedDevice.value;
  if (!device) return;
  if (confirm(`确定删除设备「${device.name}」吗？相关告警记录将一并清除。`)) {
    store.deleteDevice(device.id);
  }
}
</script>
