---
name: rn-realtime
description: Supabase realtime subscription standard for React Native apps. Use this skill whenever setting up any realtime listener — postgres_changes or broadcast. Enforces consistent setup, teardown, and channel naming to prevent memory leaks and duplicate subscriptions.
---

## The Rule

**Supabase realtime only where data must visibly sync between devices.**
If only one device writes and no other device needs to see it live — fetch once, no subscription.

---

## When to Subscribe vs Fetch Once

| Data type | Strategy |
|-----------|----------|
| Battle results (both players see instantly) | `postgres_changes` |
| Session status changes (active, ended) | `postgres_changes` |
| Participant build selections | `postgres_changes` |
| Ephemeral signals (dispute alert, presence ping) | `broadcast` |
| User's own builds list | Fetch once on mount |
| Static config / catalog data | Bundle in app, no subscription |

---

## postgres_changes Pattern

Use for data that persists in the DB and must sync between devices.

```typescript
// Inside a hook or context (not a component directly)
import { useEffect } from 'react';
import { supabase } from '@/services/supabase';

useEffect(() => {
  if (!entityId) return;  // guard: don't subscribe without a valid ID

  const channel = supabase
    .channel(`[table]:[entityId]`)   // unique channel name per entity instance
    .on(
      'postgres_changes',
      {
        event: '*',                  // INSERT | UPDATE | DELETE | * (all)
        schema: 'public',
        table: '[table_name]',
        filter: `[entity]_id=eq.${entityId}`,
      },
      (payload) => {
        // payload.eventType: 'INSERT' | 'UPDATE' | 'DELETE'
        // payload.new: new row (INSERT, UPDATE)
        // payload.old: old row (DELETE, UPDATE)
        handleRealtimeUpdate(payload);
      }
    )
    .subscribe();

  // ALWAYS unsubscribe in cleanup
  return () => {
    channel.unsubscribe();
  };
}, [entityId]);  // re-subscribe if entityId changes
```

---

## broadcast Pattern

Use for ephemeral signals — events that don't need to be persisted.

```typescript
// Sender side
const channel = supabase.channel(`session:${sessionId}`);

channel.send({
  type: 'broadcast',
  event: 'dispute',
  payload: { battleId, disputedBy: userId },
});

// Receiver side (in useEffect)
useEffect(() => {
  if (!sessionId) return;

  const channel = supabase
    .channel(`session:${sessionId}`)
    .on('broadcast', { event: 'dispute' }, (payload) => {
      handleDispute(payload);
    })
    .subscribe();

  return () => {
    channel.unsubscribe();
  };
}, [sessionId]);
```

---

## Full Hook Example

```typescript
// src/features/session/model/useSessionRealtime.ts
// Handles all realtime subscriptions for an active session.
// Called once from useSessionState — not directly from components.

import { useEffect } from 'react';
import { supabase } from '@/services/supabase';
import { mapBattleFromDTO } from '@/domain/battle/battleMappers';
import { mapParticipantFromDTO } from '@/domain/session/sessionMappers';
import type { Battle } from '@/domain/battle/Battle';
import type { SessionParticipant } from '@/domain/session/Session';

interface UseSessionRealtimeProps {
  sessionId: string | null;
  onBattleUpdate: (battle: Battle) => void;
  onParticipantUpdate: (participant: SessionParticipant) => void;
  onSessionEnd: () => void;
}

export function useSessionRealtime({
  sessionId,
  onBattleUpdate,
  onParticipantUpdate,
  onSessionEnd,
}: UseSessionRealtimeProps) {
  useEffect(() => {
    if (!sessionId) return;

    // One channel per session — subscribe to multiple tables on the same channel
    const channel = supabase
      .channel(`session:${sessionId}`)

      // Battles table changes
      .on(
        'postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: 'battles',
          filter: `session_id=eq.${sessionId}`,
        },
        (payload) => {
          if (payload.new) {
            onBattleUpdate(mapBattleFromDTO(payload.new));
          }
        }
      )

      // Participant changes (build selection)
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'session_participants',
          filter: `session_id=eq.${sessionId}`,
        },
        (payload) => {
          if (payload.new) {
            onParticipantUpdate(mapParticipantFromDTO(payload.new));
          }
        }
      )

      // Session status changes
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'sessions',
          filter: `id=eq.${sessionId}`,
        },
        (payload) => {
          if (payload.new?.status === 'ended') {
            onSessionEnd();
          }
        }
      )

      // Broadcast: dispute signal
      .on('broadcast', { event: 'dispute' }, (payload) => {
        // handle dispute signal
      })

      .subscribe();

    return () => {
      channel.unsubscribe();
    };
  }, [sessionId]);  // only sessionId in deps — callbacks are stable via useCallback
}
```

---

## Channel Naming Convention

```
[table]:[entityId]          → single table, single entity
session:[sessionId]         → session-scoped (multiple tables)
tournament:[tournamentId]   → tournament-scoped
user:[userId]               → user-scoped
```

**Always include the entity ID** — never subscribe to a whole table without a filter.

---

## Rules

1. **Always unsubscribe in `useEffect` cleanup.** No exceptions. Memory leaks cause degraded performance in long sessions.
2. **One channel per logical scope** (session, tournament). Subscribe to multiple tables on the same channel — don't create multiple channels for the same entity.
3. **Always filter.** Never subscribe to `filter: undefined` — this subscribes to the entire table.
4. **Guard with entity ID.** `if (!sessionId) return;` at the top of every effect.
5. **Map DTOs immediately** in the payload handler using the entity's mapper.
6. **Callbacks must be stable** (`useCallback`) — don't put raw functions in the `useEffect` deps array.
7. **Only the context/hook that owns the data subscribes.** Components never subscribe directly.
8. **Don't subscribe to data that doesn't need live sync.** Fetch once on mount is simpler and cheaper.
9. **Separate realtime hooks** from state hooks when subscriptions become complex — keeps files focused.
10. **Test unsubscribe:** navigate away and back — confirm no duplicate events fire.
