---
name: rn-offline
description: Offline queue and network detection standard for React Native (Expo) apps. Use this skill when implementing offline support — queue setup, network detection, retry logic, and the OFFLINE_SYNC_ENABLED flag integration. Only relevant from Phase 6 onwards.
---

## The Rule

**Offline support is behind the `OFFLINE_SYNC_ENABLED` feature flag.**
When `false`: show a blocking overlay when offline. When `true`: queue writes and sync on restore.

---

## Network Detection

```typescript
// src/services/network.ts
// NetInfo wrapper. Single source of network state.

import NetInfo, { NetInfoState } from '@react-native-community/netinfo';

export type NetworkStatus = 'online' | 'offline' | 'unknown';

export async function getNetworkStatus(): Promise<NetworkStatus> {
  const state = await NetInfo.fetch();
  if (state.isConnected === null) return 'unknown';
  return state.isConnected ? 'online' : 'offline';
}

// Subscribe to network changes
// Returns unsubscribe function — call in useEffect cleanup
export function subscribeToNetwork(
  onStatusChange: (status: NetworkStatus) => void
): () => void {
  return NetInfo.addEventListener((state: NetInfoState) => {
    if (state.isConnected === null) return;
    onStatusChange(state.isConnected ? 'online' : 'offline');
  });
}
```

---

## Offline Queue Item Type

```typescript
// src/services/offlineQueue.ts

import AsyncStorage from '@react-native-async-storage/async-storage';

const QUEUE_KEY = 'offline_queue';
const MAX_RETRIES = 5;

export type QueueItemType =
  | 'create_battle'
  | 'update_battle'
  | 'create_build'
  | 'update_build';
  // Add new types as features require them

export interface QueueItem {
  id: string;           // client-generated UUID for the queue item
  type: QueueItemType;
  payload: unknown;     // the data needed to replay the write
  timestamp: number;    // Date.now() when queued
  retries: number;      // starts at 0
}
```

---

## Queue Operations

```typescript
// src/services/offlineQueue.ts (continued)

async function getQueue(): Promise<QueueItem[]> {
  try {
    const raw = await AsyncStorage.getItem(QUEUE_KEY);
    return raw ? JSON.parse(raw) : [];
  } catch {
    return [];
  }
}

async function saveQueue(queue: QueueItem[]): Promise<void> {
  await AsyncStorage.setItem(QUEUE_KEY, JSON.stringify(queue));
}

export async function enqueue(item: Omit<QueueItem, 'retries'>): Promise<void> {
  const queue = await getQueue();
  queue.push({ ...item, retries: 0 });
  await saveQueue(queue);
}

export async function processQueue(
  handlers: Record<QueueItemType, (payload: unknown) => Promise<void>>
): Promise<void> {
  const queue = await getQueue();
  if (queue.length === 0) return;

  const remaining: QueueItem[] = [];

  for (const item of queue) {
    try {
      await handlers[item.type](item.payload);
      // Success — don't add back to queue
    } catch {
      if (item.retries < MAX_RETRIES) {
        remaining.push({ ...item, retries: item.retries + 1 });
      }
      // Exceeded MAX_RETRIES — discard silently
    }
  }

  await saveQueue(remaining);
}

export async function clearQueue(): Promise<void> {
  await AsyncStorage.removeItem(QUEUE_KEY);
}
```

---

## Offline Hook

```typescript
// src/features/offline/model/useOfflineSync.ts
// Detects network state. Triggers queue processing on restore.
// Only active when OFFLINE_SYNC_ENABLED = true.

import { useState, useEffect, useRef } from 'react';
import { FEATURE_FLAGS } from '@/config/featureFlags';
import { subscribeToNetwork, type NetworkStatus } from '@/services/network';
import { processQueue } from '@/services/offlineQueue';

export function useOfflineSync(
  handlers: Parameters<typeof processQueue>[0]
) {
  const [networkStatus, setNetworkStatus] = useState<NetworkStatus>('unknown');
  const prevStatus = useRef<NetworkStatus>('unknown');

  useEffect(() => {
    if (!FEATURE_FLAGS.OFFLINE_SYNC_ENABLED) return;

    const unsubscribe = subscribeToNetwork((status) => {
      setNetworkStatus(status);

      // Trigger sync when coming back online
      if (prevStatus.current === 'offline' && status === 'online') {
        processQueue(handlers).catch(console.error);
      }

      prevStatus.current = status;
    });

    return unsubscribe;
  }, []);

  return { networkStatus, isOffline: networkStatus === 'offline' };
}
```

---

## Blocking Overlay (when OFFLINE_SYNC_ENABLED = false)

```typescript
// src/features/offline/ui/OfflineOverlay.tsx
// Shown when offline and OFFLINE_SYNC_ENABLED is false.
// Prevents any action that requires network.

import { View, Text, StyleSheet } from 'react-native';
import { Colors, Spacing, Typography } from '@/config/theme';

export function OfflineOverlay() {
  return (
    <View style={styles.overlay}>
      <Text style={styles.title}>You're offline</Text>
      <Text style={styles.message}>
        Please reconnect to continue.
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  overlay: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: Colors.background + 'F0',  // 94% opacity
    alignItems: 'center',
    justifyContent: 'center',
    padding: Spacing.xxl,
    zIndex: 999,
  },
  title: {
    fontSize: Typography.size.xl,
    fontWeight: Typography.weight.bold,
    color: Colors.textPrimary,
    marginBottom: Spacing.sm,
  },
  message: {
    fontSize: Typography.size.md,
    color: Colors.textSecondary,
    textAlign: 'center',
  },
});
```

---

## Ambient Sync Indicator (when OFFLINE_SYNC_ENABLED = true)

```typescript
// src/features/offline/ui/SyncStatusBanner.tsx
// Non-blocking. Shows sync state without interrupting the user.

import { View, Text, StyleSheet } from 'react-native';
import { Colors, Spacing, Typography } from '@/config/theme';

interface SyncStatusBannerProps {
  isOffline: boolean;
  queueCount?: number;
}

export function SyncStatusBanner({ isOffline, queueCount = 0 }: SyncStatusBannerProps) {
  if (!isOffline && queueCount === 0) return null;

  return (
    <View style={[styles.banner, isOffline ? styles.offline : styles.syncing]}>
      <Text style={styles.text}>
        {isOffline
          ? 'Offline — changes will sync when reconnected'
          : `Syncing ${queueCount} item${queueCount !== 1 ? 's' : ''}...`}
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  banner: {
    paddingVertical: Spacing.xs,
    paddingHorizontal: Spacing.lg,
    alignItems: 'center',
  },
  offline: {
    backgroundColor: Colors.warning + '30',
  },
  syncing: {
    backgroundColor: Colors.primary + '30',
  },
  text: {
    fontSize: Typography.size.xs,
    color: Colors.textSecondary,
  },
});
```

---

## Write Pattern with Queue Fallback

```typescript
// In a feature hook — how to write with offline fallback

import * as Crypto from 'expo-crypto';
import { FEATURE_FLAGS } from '@/config/featureFlags';
import { enqueue } from '@/services/offlineQueue';
import { getNetworkStatus } from '@/services/network';
import { createBattle } from '@/domain/battle/battleRepository';

async function logBattle(battle: Battle) {
  // 1. Always update local state optimistically first
  dispatch({ type: 'ADD_BATTLE', battle });

  // 2. Check network
  const status = await getNetworkStatus();

  if (status === 'online') {
    try {
      await createBattle(battle);
    } catch {
      // Write failed — queue it
      if (FEATURE_FLAGS.OFFLINE_SYNC_ENABLED) {
        await enqueue({
          id: Crypto.randomUUID(),
          type: 'create_battle',
          payload: battle,
          timestamp: Date.now(),
        });
      }
    }
  } else if (FEATURE_FLAGS.OFFLINE_SYNC_ENABLED) {
    // Offline — queue immediately
    await enqueue({
      id: Crypto.randomUUID(),
      type: 'create_battle',
      payload: battle,
      timestamp: Date.now(),
    });
  }
}
```

---

## Rules

1. **Gate everything behind `OFFLINE_SYNC_ENABLED`.** When `false`, show blocking overlay. No partial offline support.
2. **Optimistic UI first** — state updates before any network check.
3. **FIFO queue** — items processed in order. Never reorder.
4. **Max 5 retries per item.** Discard silently after — don't surface stale errors to users.
5. **Queue survives app restarts** — stored in AsyncStorage, not memory.
6. **One queue key** — `offline_queue`. Never split into multiple keys.
7. **Process queue on network restore only** — not on a timer.
8. **Blocking overlay vs ambient banner** — flag off = blocking, flag on = ambient. Never reverse this.
9. **`network.ts` is the only place that imports NetInfo** — everything else calls `network.ts`.
10. **Test offline mode** with airplane mode on a physical device — simulators are unreliable for network testing.
