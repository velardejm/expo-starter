---
name: rn-component
description: Reusable component standards for React Native (Expo) apps. Use this skill whenever creating any UI component — whether in a feature's ui/ folder or in shared/components/. Enforces consistent file structure, prop typing, theme usage, and common patterns for loading, empty, and error states.
---

## Component File Structure

Every component file follows this exact order:

```typescript
// 1. Imports (external first, then internal)
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { Colors, Spacing, Typography, Radius } from '@/config/theme';

// 2. Types / interfaces (props first, then any local types)
interface ExampleCardProps {
  title: string;
  subtitle?: string;
  onPress: () => void;
  isDisabled?: boolean;
}

// 3. Component function
export function ExampleCard({
  title,
  subtitle,
  onPress,
  isDisabled = false,
}: ExampleCardProps) {
  return (
    <TouchableOpacity
      style={[styles.card, isDisabled && styles.cardDisabled]}
      onPress={onPress}
      disabled={isDisabled}
      activeOpacity={0.7}
    >
      <Text style={styles.title}>{title}</Text>
      {subtitle && <Text style={styles.subtitle}>{subtitle}</Text>}
    </TouchableOpacity>
  );
}

// 4. Styles (always at the bottom)
const styles = StyleSheet.create({
  card: {
    backgroundColor: Colors.surface,
    borderRadius: Radius.lg,
    padding: Spacing.lg,
  },
  cardDisabled: {
    opacity: 0.4,
  },
  title: {
    fontSize: Typography.size.lg,
    fontWeight: Typography.weight.semibold,
    color: Colors.textPrimary,
  },
  subtitle: {
    fontSize: Typography.size.sm,
    color: Colors.textSecondary,
    marginTop: Spacing.xs,
  },
});
```

---

## Where Components Live

| Component type | Location | Rule |
|---------------|----------|------|
| Used by 1 feature only | `src/features/[feature]/ui/` | Feature-private |
| Used by 2+ features | `src/shared/components/` | Move here when second use appears |
| Screen (full page) | `src/features/[feature]/ui/[Name]Screen.tsx` | Always suffixed `Screen` |
| Modal | `src/features/[feature]/ui/[Name]Modal.tsx` | Always suffixed `Modal` |
| Panel (partial page section) | `src/features/[feature]/ui/[Name]Panel.tsx` | Always suffixed `Panel` |
| Small reusable element | `src/shared/components/[Name].tsx` | No suffix needed |

---

## Standard Shared Components

These must be created in Phase 1 and used consistently:

### `src/shared/components/LoadingScreen.tsx`
```typescript
import { View, ActivityIndicator, StyleSheet } from 'react-native';
import { Colors } from '@/config/theme';

export function LoadingScreen() {
  return (
    <View style={styles.container}>
      <ActivityIndicator size="large" color={Colors.primary} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: Colors.background,
  },
});
```

### `src/shared/components/EmptyState.tsx`
```typescript
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { Colors, Spacing, Typography, Radius } from '@/config/theme';

interface EmptyStateProps {
  message: string;
  actionLabel?: string;
  onAction?: () => void;
}

export function EmptyState({ message, actionLabel, onAction }: EmptyStateProps) {
  return (
    <View style={styles.container}>
      <Text style={styles.message}>{message}</Text>
      {actionLabel && onAction && (
        <TouchableOpacity style={styles.button} onPress={onAction}>
          <Text style={styles.buttonText}>{actionLabel}</Text>
        </TouchableOpacity>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: Spacing.xxl,
  },
  message: {
    fontSize: Typography.size.md,
    color: Colors.textSecondary,
    textAlign: 'center',
    marginBottom: Spacing.xl,
  },
  button: {
    backgroundColor: Colors.primary,
    borderRadius: Radius.md,
    paddingVertical: Spacing.md,
    paddingHorizontal: Spacing.xl,
  },
  buttonText: {
    color: Colors.textPrimary,
    fontSize: Typography.size.md,
    fontWeight: Typography.weight.semibold,
  },
});
```

### `src/shared/components/ErrorMessage.tsx`
```typescript
import { View, Text, StyleSheet } from 'react-native';
import { Colors, Spacing, Typography, Radius } from '@/config/theme';

interface ErrorMessageProps {
  message: string;
}

export function ErrorMessage({ message }: ErrorMessageProps) {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>{message}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    backgroundColor: Colors.danger + '20',   // 12% opacity
    borderRadius: Radius.md,
    padding: Spacing.md,
    marginVertical: Spacing.sm,
  },
  text: {
    color: Colors.danger,
    fontSize: Typography.size.sm,
  },
});
```

### `src/shared/components/PrimaryButton.tsx`
```typescript
import { TouchableOpacity, Text, ActivityIndicator, StyleSheet } from 'react-native';
import { Colors, Spacing, Typography, Radius } from '@/config/theme';

interface PrimaryButtonProps {
  label: string;
  onPress: () => void;
  isDisabled?: boolean;
  isLoading?: boolean;
  variant?: 'primary' | 'danger' | 'ghost';
}

export function PrimaryButton({
  label,
  onPress,
  isDisabled = false,
  isLoading = false,
  variant = 'primary',
}: PrimaryButtonProps) {
  return (
    <TouchableOpacity
      style={[styles.button, styles[variant], (isDisabled || isLoading) && styles.disabled]}
      onPress={onPress}
      disabled={isDisabled || isLoading}
      activeOpacity={0.8}
    >
      {isLoading ? (
        <ActivityIndicator color={Colors.textPrimary} size="small" />
      ) : (
        <Text style={styles.label}>{label}</Text>
      )}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  button: {
    borderRadius: Radius.md,
    paddingVertical: Spacing.md,
    paddingHorizontal: Spacing.xl,
    alignItems: 'center',
    justifyContent: 'center',
    minHeight: 48,
  },
  primary: {
    backgroundColor: Colors.primary,
  },
  danger: {
    backgroundColor: Colors.danger,
  },
  ghost: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: Colors.border,
  },
  disabled: {
    opacity: 0.4,
  },
  label: {
    color: Colors.textPrimary,
    fontSize: Typography.size.md,
    fontWeight: Typography.weight.semibold,
  },
});
```

---

## Rules

1. **Props interface always named `[ComponentName]Props`** and defined directly above the component.
2. **Destructure props** in the function signature — never access via `props.x`.
3. **Default props** via destructuring defaults, not `defaultProps`.
4. **Named exports only** — no default exports for components (except screen route files in `app/`).
5. **`activeOpacity={0.7}`** on all `TouchableOpacity` elements.
6. **`minHeight: 48`** on all tappable elements (accessibility minimum touch target).
7. **Never put styles inline** except for truly dynamic values.
8. **`StyleSheet.create()`** always — never raw objects.
9. **Import shared components from barrel:** `import { EmptyState } from '@/shared/components'`
10. **Shared `components/index.ts` barrel** must be updated when a new shared component is added.

---

## Shared Components Barrel

```typescript
// src/shared/components/index.ts
export { LoadingScreen } from './LoadingScreen';
export { EmptyState } from './EmptyState';
export { ErrorMessage } from './ErrorMessage';
export { PrimaryButton } from './PrimaryButton';
// Add new shared components here
```
