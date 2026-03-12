---
name: rn-styles
description: Centralized styling system for React Native (Expo) apps. Use this skill whenever creating or modifying any styles, colors, spacing, typography, or theme tokens. All styling must flow from the central theme file — never hardcode values.
---

## The Rule

**Never hardcode a color, spacing value, font size, or border radius anywhere in the app.**
All values come from `src/config/theme.ts`. No exceptions.

---

## File: `src/config/theme.ts`

This file must be created in Phase 1 and never modified without user approval of the change.

```typescript
// src/config/theme.ts
// Central design token registry.
// Import this wherever styles are needed. Never import from anywhere else.

export const Colors = {
  // Primary
  primary: '#007AFF',
  primaryDark: '#0056CC',
  primaryLight: '#3395FF',

  // Semantic
  success: '#34C759',
  warning: '#FF9500',
  danger: '#FF3B30',
  info: '#5AC8FA',

  // Neutrals
  background: '#000000',
  surface: '#1C1C1E',
  surfaceRaised: '#2C2C2E',
  border: '#38383A',

  // Text
  textPrimary: '#FFFFFF',
  textSecondary: '#EBEBF5CC',  // 80% white
  textTertiary: '#EBEBF54D',   // 30% white
  textDisabled: '#EBEBF530',   // 19% white
  textInverse: '#000000',

  // Interactive
  buttonPrimary: '#007AFF',
  buttonDestructive: '#FF3B30',
  buttonDisabled: '#38383A',
} as const;

export const Spacing = {
  xs: 4,
  sm: 8,
  md: 12,
  lg: 16,
  xl: 24,
  xxl: 32,
  xxxl: 48,
} as const;

export const Typography = {
  // Font sizes
  size: {
    xs: 11,
    sm: 13,
    md: 15,
    lg: 17,
    xl: 20,
    xxl: 24,
    xxxl: 32,
    display: 40,
  },
  // Font weights
  weight: {
    regular: '400' as const,
    medium: '500' as const,
    semibold: '600' as const,
    bold: '700' as const,
    heavy: '800' as const,
  },
  // Line heights
  lineHeight: {
    tight: 1.2,
    normal: 1.4,
    relaxed: 1.6,
  },
} as const;

export const Radius = {
  sm: 6,
  md: 10,
  lg: 14,
  xl: 20,
  full: 9999,
} as const;

export const Shadows = {
  sm: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 2,
  },
  md: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.3,
    shadowRadius: 8,
    elevation: 5,
  },
  lg: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 8 },
    shadowOpacity: 0.4,
    shadowRadius: 16,
    elevation: 10,
  },
} as const;

// Convenience re-export for common imports
export const Theme = {
  Colors,
  Spacing,
  Typography,
  Radius,
  Shadows,
} as const;
```

---

## Usage Pattern

```typescript
// In any component file
import { StyleSheet } from 'react-native';
import { Colors, Spacing, Typography, Radius } from '@/config/theme';

const styles = StyleSheet.create({
  container: {
    backgroundColor: Colors.surface,
    padding: Spacing.lg,
    borderRadius: Radius.md,
  },
  title: {
    fontSize: Typography.size.xl,
    fontWeight: Typography.weight.bold,
    color: Colors.textPrimary,
  },
  subtitle: {
    fontSize: Typography.size.md,
    color: Colors.textSecondary,
    marginTop: Spacing.xs,
  },
});
```

---

## Rules

1. **Import from `@/config/theme` only.** Never from relative paths like `../../config/theme`.
2. **No inline styles** except for truly dynamic values (e.g., animated transforms, width from state).
3. **No hardcoded hex colors** anywhere in component files.
4. **No hardcoded numbers** for spacing, font size, or border radius.
5. **`StyleSheet.create()`** must be used for all static styles — never plain objects.
6. **Styles go at the bottom** of every component file, after the component function.
7. **When adding a new color or token**, add it to `theme.ts` first, then use it. Never create a one-off value.
8. **Dark mode:** The default theme is dark. If light mode is needed, add a `LightColors` export to `theme.ts` — never duplicate the file.

---

## Naming Conventions

```typescript
// Style keys: camelCase, descriptive
const styles = StyleSheet.create({
  container: {},        // wrapper/root element
  header: {},           // top section
  title: {},            // primary text
  subtitle: {},         // secondary text
  body: {},             // content area
  footer: {},           // bottom section
  row: {},              // horizontal flex container
  card: {},             // elevated surface
  badge: {},            // small indicator
  button: {},           // interactive element
  buttonText: {},       // text inside button
  emptyState: {},       // empty/zero state container
  emptyStateText: {},   // empty state message
});
```

---

## Common Patterns

### Card surface
```typescript
card: {
  backgroundColor: Colors.surface,
  borderRadius: Radius.lg,
  padding: Spacing.lg,
  ...Shadows.md,
},
```

### Full-screen container
```typescript
container: {
  flex: 1,
  backgroundColor: Colors.background,
},
```

### Centered content
```typescript
centered: {
  flex: 1,
  alignItems: 'center',
  justifyContent: 'center',
  backgroundColor: Colors.background,
},
```

### Primary button
```typescript
button: {
  backgroundColor: Colors.buttonPrimary,
  borderRadius: Radius.md,
  paddingVertical: Spacing.md,
  paddingHorizontal: Spacing.xl,
  alignItems: 'center',
},
buttonText: {
  color: Colors.textPrimary,
  fontSize: Typography.size.md,
  fontWeight: Typography.weight.semibold,
},
buttonDisabled: {
  backgroundColor: Colors.buttonDisabled,
},
```

### Separator / divider
```typescript
separator: {
  height: 1,
  backgroundColor: Colors.border,
  marginVertical: Spacing.sm,
},
```
