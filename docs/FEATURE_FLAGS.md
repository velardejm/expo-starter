# Feature Flags
# [APP_NAME]

> Feature flags are the primary mechanism for developing features in isolation.
> A flag is a single boolean. `false` = feature hidden/blocked. `true` = feature active.
> Flip one line to enable or disable a feature. No code changes needed.

---

## The Control Panel

**File:** `src/config/featureFlags.ts`

```typescript
export const FEATURE_FLAGS = {
  // --- CORE EXTENSIONS ---
  // [FLAG_NAME]: false,

  // --- ADDITIONAL MODES ---
  // [FLAG_NAME]: false,

  // --- FUTURE ---
  // [FLAG_NAME]: false,
} as const;
```

---

## Flag Reference

### `[FLAG_NAME]`

| Property | Value |
|----------|-------|
| Default | `false` |
| Enable when | [condition — e.g., "after Phase 5 is confirmed stable"] |

**When `false`:**
- [What the user sees / what is hidden]

**When `true`:**
- [What becomes available]

**Controls:**
- `src/features/[feature]/` — the entire feature module
- [Specific components or files gated by this flag]

---

## How to Use Flags in Code

### Hide a screen or button
```typescript
import { FEATURE_FLAGS } from '@/config/featureFlags';

{FEATURE_FLAGS.FLAG_NAME && (
  <TouchableOpacity onPress={goToFeature}>
    <Text>Feature</Text>
  </TouchableOpacity>
)}
```

### Guard an entire screen
```typescript
export default function FeatureScreen() {
  if (!FEATURE_FLAGS.FLAG_NAME) return null;
  return <FeatureView />;
}
```

### Gate logic inside a hook
```typescript
function useScore(data: DataType[]) {
  if (FEATURE_FLAGS.FLAG_NAME) {
    return computeAdvancedScore(data);
  }
  return computeBasicScore(data);
}
```

---

## Rules

1. **Default is always `false`.** New flags start disabled.
2. **One flag at a time.** Only enable one flag per development session.
3. **Flags are build-time.** No remote config. No runtime toggles. Flip → rebuild.
4. **Claude checks flags before adding entry points.**
5. **Remove flags when stable.** Delete the constant + all conditional code referencing it.
6. **Never modify a flag file when debugging unrelated features.**

---

## Flag Lifecycle

```
PLANNED → add flag (default: false)
   ↓
BUILDING → develop behind flag; stays false in main
   ↓
TESTING → flip true locally; run manual test plan
   ↓
STABLE → flag stays true in production for one release cycle
   ↓
REMOVED → delete constant + all conditional code
```
