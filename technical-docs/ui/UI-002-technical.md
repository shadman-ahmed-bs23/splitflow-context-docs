# UI-002: Real-Time Updates - Technical Specification

**Feature ID:** UI-002  
**Category:** User Interface  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. Frontend Implementation

### 1.1 WebSocket Client Setup
**File:** `plugins/websocket.client.ts`

```typescript
export default defineNuxtPlugin(() => {
  const { $io } = useNuxtApp();
  
  // Initialize Socket.io client
  const socket = io(useRuntimeConfig().public.wsUrl, {
    auth: {
      token: useCookie('access_token').value,
    },
    transports: ['websocket', 'polling'],
  });
  
  // Connection events
  socket.on('connect', () => {
    console.log('WebSocket connected');
  });
  
  socket.on('disconnect', () => {
    console.log('WebSocket disconnected');
  });
  
  return {
    provide: {
      socket,
    },
  };
});
```

---

### 1.2 Composable: useRealtime
**File:** `composables/useRealtime.ts`

```typescript
export const useRealtime = () => {
  const { $socket } = useNuxtApp();
  const isConnected = ref(false);
  
  const connect = () => {
    $socket.connect();
    isConnected.value = $socket.connected;
  };
  
  const disconnect = () => {
    $socket.disconnect();
    isConnected.value = false;
  };
  
  const subscribe = (channel: string, callback: Function) => {
    $socket.on(channel, callback);
    
    return () => {
      $socket.off(channel, callback);
    };
  };
  
  const unsubscribe = (channel: string) => {
    $socket.off(channel);
  };
  
  return {
    isConnected,
    connect,
    disconnect,
    subscribe,
    unsubscribe,
  };
};
```

---

## 2. Real-Time Update Implementation

### 2.1 Balance Updates
**File:** `composables/useGroupBalance.ts`

```typescript
export const useGroupBalance = (groupId: string) => {
  const balance = ref('0.00');
  const { subscribe, isConnected } = useRealtime();
  
  // Subscribe to group balance updates
  onMounted(() => {
    const unsubscribe = subscribe(`group.${groupId}.balance`, (data: any) => {
      balance.value = data.main_balance;
    });
    
    onUnmounted(() => {
      unsubscribe();
    });
  });
  
  // Fallback to polling if WebSocket disconnected
  watch(isConnected, (connected) => {
    if (!connected) {
      // Start polling
      const interval = setInterval(async () => {
        const { data } = await $fetch(`/api/v1/groups/${groupId}/balance`);
        balance.value = data.main_balance;
      }, 10000);
      
      // Stop polling when reconnected
      watchOnce(isConnected, () => {
        clearInterval(interval);
      });
    }
  });
  
  return { balance };
};
```

---

### 2.2 Unread Count Updates
**File:** `composables/useUnreadCounts.ts`

```typescript
export const useUnreadCounts = () => {
  const unreadCounts = ref<Record<string, number>>({});
  const { subscribe } = useRealtime();
  
  onMounted(() => {
    subscribe('user.unread_counts', (data: any) => {
      unreadCounts.value = data.counts;
    });
  });
  
  return { unreadCounts };
};
```

---

## 3. Fallback Strategy

### 3.1 Polling Implementation
```typescript
const usePolling = (endpoint: string, interval: number = 10000) => {
  const data = ref(null);
  let pollInterval: NodeJS.Timeout | null = null;
  
  const startPolling = async () => {
    const fetchData = async () => {
      try {
        const response = await $fetch(endpoint);
        data.value = response;
      } catch (error) {
        console.error('Polling error:', error);
      }
    };
    
    await fetchData();
    pollInterval = setInterval(fetchData, interval);
  };
  
  const stopPolling = () => {
    if (pollInterval) {
      clearInterval(pollInterval);
      pollInterval = null;
    }
  };
  
  onUnmounted(() => {
    stopPolling();
  });
  
  return { data, startPolling, stopPolling };
};
```

---

## 4. Connection Status Indicator

### 4.1 Component: ConnectionStatus
**File:** `components/ConnectionStatus.vue`

```vue
<template>
  <div class="connection-status" :class="{ 'disconnected': !isConnected }">
    <span v-if="isConnected">● Connected</span>
    <span v-else>○ Disconnected (Polling)</span>
  </div>
</template>

<script setup>
const { isConnected } = useRealtime();
</script>

<style scoped>
.connection-status {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 8px 12px;
  background: #10B981;
  color: white;
  border-radius: 4px;
  font-size: 12px;
}

.disconnected {
  background: #F59E0B;
}
</style>
```

---

## 5. Optimistic Updates

### 5.1 Pattern
```typescript
const submitExpense = async (data: ExpenseData) => {
  // Optimistic update
  const optimisticExpense = {
    id: 'temp-' + Date.now(),
    ...data,
    status: 'pending',
  };
  
  expenses.value.push(optimisticExpense);
  
  try {
    const response = await $fetch('/api/v1/expenses', {
      method: 'POST',
      body: data,
    });
    
    // Replace optimistic with real data
    const index = expenses.value.findIndex(e => e.id === optimisticExpense.id);
    expenses.value[index] = response.data;
  } catch (error) {
    // Rollback optimistic update
    expenses.value = expenses.value.filter(e => e.id !== optimisticExpense.id);
    throw error;
  }
};
```

---

## 6. Error Handling

### 6.1 Reconnection Logic
```typescript
const useReconnection = () => {
  const { $socket } = useNuxtApp();
  let reconnectAttempts = 0;
  const maxAttempts = 5;
  
  $socket.on('disconnect', () => {
    if (reconnectAttempts < maxAttempts) {
      const delay = Math.min(1000 * Math.pow(2, reconnectAttempts), 30000);
      setTimeout(() => {
        reconnectAttempts++;
        $socket.connect();
      }, delay);
    }
  });
  
  $socket.on('connect', () => {
    reconnectAttempts = 0;
  });
};
```

---

## 7. Testing Strategy

### 7.1 Unit Tests
- WebSocket connection
- Fallback to polling
- Optimistic updates

### 7.2 Integration Tests
- Real-time updates
- Connection handling
- Error recovery

---

## 8. Dependencies

- Socket.io client
- Nuxt 3
- WebSocket server (backend)

---

## 9. Related Documentation

- [NOTIF-001 Technical Spec](../notifications/NOTIF-001-technical.md) - Notifications
- [UI-001 Technical Spec](./UI-001-technical.md) - Mobile Design
