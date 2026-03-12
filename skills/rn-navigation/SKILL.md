---
name: rn-navigation
description: Tab + stack navigation standard for React Native (Expo Router) apps. Use this skill whenever creating or modifying navigation structure, layouts, tab bars, or route files. All navigation follows the Expo Router file-based pattern with tabs as the root navigator.
---

## Navigation Architecture

This app uses **Expo Router** with a **tabs-first** structure.
Tabs are the root navigator. Stacks live inside each tab.

```
app/
  _layout.tsx              Root layout — providers + root Stack
  (tabs)/
    _layout.tsx            Tab navigator — defines all tabs
    index.tsx              Tab 1: Home (first tab, always index)
    [tab2].tsx             Tab 2
    [tab3].tsx             Tab 3
  [feature]/
    index.tsx              Feature list screen (pushed from a tab)
    create.tsx             Feature create screen
    [id].tsx               Feature detail/edit screen
```

---

## Root Layout — `app/_layout.tsx`

```typescript
// app/_layout.tsx
// Root layout. Providers wrap the entire app.
// Order matters — outer providers have no dependencies.

import 'react-native-url-polyfill/auto';
import { Stack } from 'expo-router';

// Context Provider Hierarchy — dependency order matters
// 1. UserProvider         [ACTIVE] — identity, no deps
// 2. DataProvider         [ACTIVE] — depends on UserProvider
// 3. FeatureProvider      [ACTIVE] — depends on User + Data
// 4. FlaggedProvider      [Phase N, flagged] — enable with FLAG_NAME

import { UserProvider } from '@/features/auth/model/UserProvider';

export default function RootLayout() {
  return (
    <UserProvider>
      <Stack screenOptions={{ headerShown: false }} />
    </UserProvider>
  );
}
```

---

## Tab Layout — `app/(tabs)/_layout.tsx`

```typescript
// app/(tabs)/_layout.tsx
// Defines the tab bar. Add/remove tabs here only.
// Flagged tabs must check their feature flag before rendering.

import { Tabs } from 'expo-router';
import { Colors } from '@/config/theme';
import { FEATURE_FLAGS } from '@/config/featureFlags';

// Import tab bar icons (use expo/vector-icons or custom SVGs)
import { Ionicons } from '@expo/vector-icons';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        headerShown: false,
        tabBarActiveTintColor: Colors.primary,
        tabBarInactiveTintColor: Colors.textTertiary,
        tabBarStyle: {
          backgroundColor: Colors.surface,
          borderTopColor: Colors.border,
          borderTopWidth: 1,
        },
        tabBarLabelStyle: {
          fontSize: 10,
          fontWeight: '500',
        },
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home-outline" color={color} size={size} />
          ),
        }}
      />

      <Tabs.Screen
        name="[tab2]"
        options={{
          title: '[Tab 2]',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="list-outline" color={color} size={size} />
          ),
        }}
      />

      {/* Flagged tab example — only rendered when flag is true */}
      {FEATURE_FLAGS.FEATURE_NAME && (
        <Tabs.Screen
          name="[flagged-tab]"
          options={{
            title: '[Feature]',
            tabBarIcon: ({ color, size }) => (
              <Ionicons name="trophy-outline" color={color} size={size} />
            ),
          }}
        />
      )}

      <Tabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person-outline" color={color} size={size} />
          ),
        }}
      />
    </Tabs>
  );
}
```

---

## Screen File Pattern (Thin Routes)

Route files in `app/` are thin wrappers only. No logic, no state, no styles.

```typescript
// app/(tabs)/[feature]/index.tsx
// Thin route — passes params to the feature screen component.

import { useLocalSearchParams } from 'expo-router';
import { FeatureListScreen } from '@/features/[feature]/ui/FeatureListScreen';

export default function FeatureListRoute() {
  return <FeatureListScreen />;
}
```

```typescript
// app/(tabs)/[feature]/[id].tsx
// Thin route with param extraction.

import { useLocalSearchParams } from 'expo-router';
import { FeatureDetailScreen } from '@/features/[feature]/ui/FeatureDetailScreen';

export default function FeatureDetailRoute() {
  const { id } = useLocalSearchParams<{ id: string }>();
  return <FeatureDetailScreen id={id} />;
}
```

---

## Navigation from Components

```typescript
import { router } from 'expo-router';

// Push a screen
router.push('/feature/create');

// Push with params
router.push(`/feature/${id}`);

// Replace (no back button)
router.replace('/');

// Go back
router.back();
```

---

## Folder Rules

| Folder | Purpose | Who creates screens here |
|--------|---------|--------------------------|
| `app/(tabs)/` | Tab root screens | lead-orchestrator plans; react-native-specialist implements |
| `app/[feature]/` | Feature stack screens (pushed from tabs) | react-native-specialist |
| `src/features/[feature]/ui/` | Actual screen components | react-native-specialist |

**Thin route files live in `app/`. Screen logic lives in `src/features/`.**

---

## Naming Conventions

| Route | File | Screen component |
|-------|------|-----------------|
| Home tab | `app/(tabs)/index.tsx` | `HomeScreen.tsx` |
| Feature list | `app/(tabs)/[feature]/index.tsx` | `[Feature]ListScreen.tsx` |
| Feature create | `app/[feature]/create.tsx` | `Create[Feature]Screen.tsx` |
| Feature detail | `app/[feature]/[id].tsx` | `[Feature]DetailScreen.tsx` |
| Feature edit | `app/[feature]/[id]/edit.tsx` | `Edit[Feature]Screen.tsx` |

---

## Header Configuration

```typescript
// Show a header on a specific screen
<Stack.Screen
  options={{
    headerShown: true,
    title: 'Screen Title',
    headerStyle: { backgroundColor: Colors.surface },
    headerTintColor: Colors.textPrimary,
    headerTitleStyle: {
      fontWeight: Typography.weight.semibold,
      fontSize: Typography.size.lg,
    },
  }}
/>
```

---

## Rules

1. **Tabs are the root.** Never put a stack as the root navigator.
2. **Thin routes only in `app/`.** No business logic, hooks, or state in route files.
3. **Flagged tabs use feature flag check** before rendering the `<Tabs.Screen>`.
4. **All navigation uses `router` from `expo-router`** — never `useNavigation` from React Navigation directly.
5. **Params are always strings** in Expo Router — parse/validate in the screen component, not the route.
6. **Tab count:** 3–5 tabs. More than 5 = reconsider the IA.
7. **`index.tsx` is always the first (default) tab.**
8. **Add the tab to `(tabs)/_layout.tsx` before creating the screen file** — routing must exist before it's linked.
