---
name: rn-animations
description: Animation standard for React Native (Expo) apps using react-native-reanimated. Use this skill whenever adding any animation — transitions, countdowns, feedback, layout changes. Keeps animations simple, purposeful, and consistent.
---

## The Rule

**Animations serve the user, not the developer.**
Every animation must have a reason: feedback, orientation, or delight. No animations for their own sake.

---

## When to Animate vs Not

| Use case | Animate? |
|----------|----------|
| Button press feedback | ✅ Yes — scale down slightly |
| Countdown timer bar | ✅ Yes — width shrinks over time |
| Screen transitions | ✅ Yes — Expo Router handles this automatically |
| Error shake | ✅ Yes — horizontal shake on invalid submit |
| Banner slide in | ✅ Yes — slide from top/bottom |
| Card appear in list | ⚠️ Only if list is small — never animate 50+ items |
| Loading spinner | ✅ Yes — use ActivityIndicator (built-in) |
| Color change on state | ✅ Yes — interpolate between colors |
| Complex parallax / scroll effects | ❌ No — adds complexity, rarely justified |
| Page-level transitions | ❌ No — Expo Router handles this |

---

## Setup

```typescript
// Already installed with Expo — ensure it's in app.json plugins if needed
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
  withRepeat,
  withSequence,
  Easing,
  interpolate,
  runOnJS,
} from 'react-native-reanimated';
```

---

## Button Press Feedback

```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';
import { Pressable } from 'react-native';

export function AnimatedButton({ onPress, children, style }: AnimatedButtonProps) {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <Pressable
      onPressIn={() => { scale.value = withSpring(0.95); }}
      onPressOut={() => { scale.value = withSpring(1); }}
      onPress={onPress}
    >
      <Animated.View style={[style, animatedStyle]}>
        {children}
      </Animated.View>
    </Pressable>
  );
}
```

---

## Countdown Timer Bar

```typescript
import { useEffect } from 'react';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  Easing,
  runOnJS,
} from 'react-native-reanimated';
import { View, StyleSheet } from 'react-native';
import { Colors, Radius } from '@/config/theme';

interface CountdownBarProps {
  durationMs: number;       // total duration in milliseconds
  onComplete: () => void;
}

export function CountdownBar({ durationMs, onComplete }: CountdownBarProps) {
  const progress = useSharedValue(1);  // 1 = full, 0 = empty

  useEffect(() => {
    progress.value = withTiming(0, {
      duration: durationMs,
      easing: Easing.linear,
    }, (finished) => {
      if (finished) runOnJS(onComplete)();
    });

    return () => {
      progress.value = 1;  // reset on unmount
    };
  }, [durationMs]);

  const barStyle = useAnimatedStyle(() => ({
    width: `${progress.value * 100}%`,
  }));

  return (
    <View style={styles.track}>
      <Animated.View style={[styles.bar, barStyle]} />
    </View>
  );
}

const styles = StyleSheet.create({
  track: {
    height: 4,
    backgroundColor: Colors.border,
    borderRadius: Radius.full,
    overflow: 'hidden',
  },
  bar: {
    height: '100%',
    backgroundColor: Colors.primary,
    borderRadius: Radius.full,
  },
});
```

---

## Slide-In Banner

```typescript
import { useEffect } from 'react';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withDelay,
} from 'react-native-reanimated';
import { StyleSheet } from 'react-native';
import { Colors, Spacing } from '@/config/theme';

interface SlideBannerProps {
  visible: boolean;
  children: React.ReactNode;
}

export function SlideBanner({ visible, children }: SlideBannerProps) {
  const translateY = useSharedValue(-80);
  const opacity = useSharedValue(0);

  useEffect(() => {
    if (visible) {
      translateY.value = withTiming(0, { duration: 300 });
      opacity.value = withTiming(1, { duration: 300 });
    } else {
      translateY.value = withTiming(-80, { duration: 250 });
      opacity.value = withTiming(0, { duration: 250 });
    }
  }, [visible]);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: translateY.value }],
    opacity: opacity.value,
  }));

  return (
    <Animated.View style={[styles.banner, animatedStyle]}>
      {children}
    </Animated.View>
  );
}

const styles = StyleSheet.create({
  banner: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    padding: Spacing.md,
    backgroundColor: Colors.surface,
    zIndex: 10,
  },
});
```

---

## Error Shake

```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSequence,
  withTiming,
} from 'react-native-reanimated';

export function useShakeAnimation() {
  const translateX = useSharedValue(0);

  const shake = () => {
    translateX.value = withSequence(
      withTiming(-8, { duration: 60 }),
      withTiming(8, { duration: 60 }),
      withTiming(-8, { duration: 60 }),
      withTiming(8, { duration: 60 }),
      withTiming(0, { duration: 60 }),
    );
  };

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }));

  return { shake, animatedStyle };
}

// Usage in a form:
// const { shake, animatedStyle } = useShakeAnimation();
// <Animated.View style={animatedStyle}>
//   <TextInput ... />
// </Animated.View>
// Call shake() when validation fails
```

---

## Fade In on Mount

```typescript
import { useEffect } from 'react';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
} from 'react-native-reanimated';

export function FadeIn({
  children,
  delay = 0,
}: {
  children: React.ReactNode;
  delay?: number;
}) {
  const opacity = useSharedValue(0);

  useEffect(() => {
    opacity.value = withTiming(1, { duration: 300 });
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({ opacity: opacity.value }));

  return (
    <Animated.View style={animatedStyle}>
      {children}
    </Animated.View>
  );
}
```

---

## Rules

1. **`useSharedValue` + `useAnimatedStyle`** — always. Never use the old `Animated.Value` API.
2. **`runOnJS(callback)()`** to call React state setters from animation callbacks — required by Reanimated's worklet thread.
3. **Durations:** enter = 250–350ms, exit = 200–250ms, feedback = 60–150ms. Never longer.
4. **`withSpring` for physical interactions** (button press, drag). **`withTiming` for everything else.**
5. **Clean up on unmount** — reset shared values in `useEffect` return function.
6. **Never animate 50+ items simultaneously** — causes frame drops.
7. **Never block interaction** — animations are cosmetic, not functional gates.
8. **Extract animation logic into hooks** (`useShakeAnimation`, `useCountdown`) — keeps component files clean.
9. **`Easing.linear` for progress bars.** `Easing.out(Easing.ease)` for most other `withTiming` calls.
10. **Test on a physical device** — simulator frame rates are misleading.
