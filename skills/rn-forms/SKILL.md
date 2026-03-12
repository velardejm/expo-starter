---
name: rn-forms
description: Form handling standard for React Native (Expo) apps. Use this skill whenever building any form — inputs, validation, submit logic, error display, or keyboard handling. No external form libraries. Plain useState + StyleSheet only.
---

## The Rule

**No react-hook-form, Formik, or any external form library.**
Forms use plain `useState`. Validation runs on submit. Errors display inline.

---

## Standard Form Pattern

```typescript
// src/features/[feature]/ui/[Name]Form.tsx

import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  ScrollView,
  StyleSheet,
} from 'react-native';
import { Colors, Spacing, Typography, Radius } from '@/config/theme';

// 1. Field state type
type FormFields = {
  name: string;
  email: string;
};

type FormErrors = Partial<Record<keyof FormFields, string>>;

// 2. Validation function — pure, no side effects
function validate(fields: FormFields): FormErrors {
  const errors: FormErrors = {};

  if (!fields.name.trim()) {
    errors.name = 'Name is required';
  }

  if (!fields.email.trim()) {
    errors.email = 'Email is required';
  } else if (!fields.email.includes('@')) {
    errors.email = 'Enter a valid email';
  }

  return errors;
}

// 3. Component
interface ExampleFormProps {
  onSubmit: (fields: FormFields) => Promise<void>;
  isLoading?: boolean;
}

export function ExampleForm({ onSubmit, isLoading = false }: ExampleFormProps) {
  const [fields, setFields] = useState<FormFields>({ name: '', email: '' });
  const [errors, setErrors] = useState<FormErrors>({});
  const [submitError, setSubmitError] = useState<string | null>(null);

  function updateField(key: keyof FormFields, value: string) {
    setFields(prev => ({ ...prev, [key]: value }));
    // Clear field error on change
    if (errors[key]) {
      setErrors(prev => ({ ...prev, [key]: undefined }));
    }
  }

  async function handleSubmit() {
    const validationErrors = validate(fields);
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }

    setSubmitError(null);

    try {
      await onSubmit(fields);
    } catch (err) {
      setSubmitError('Something went wrong. Please try again.');
    }
  }

  const isSubmitDisabled = isLoading || !fields.name.trim() || !fields.email.trim();

  return (
    <KeyboardAvoidingView
      style={styles.keyboardAvoid}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <ScrollView
        contentContainerStyle={styles.scrollContent}
        keyboardShouldPersistTaps="handled"
      >
        {/* Field */}
        <View style={styles.fieldGroup}>
          <Text style={styles.label}>Name</Text>
          <TextInput
            style={[styles.input, errors.name ? styles.inputError : null]}
            value={fields.name}
            onChangeText={v => updateField('name', v)}
            placeholder="Your name"
            placeholderTextColor={Colors.textTertiary}
            autoCapitalize="words"
            returnKeyType="next"
          />
          {errors.name && <Text style={styles.fieldError}>{errors.name}</Text>}
        </View>

        {/* Field */}
        <View style={styles.fieldGroup}>
          <Text style={styles.label}>Email</Text>
          <TextInput
            style={[styles.input, errors.email ? styles.inputError : null]}
            value={fields.email}
            onChangeText={v => updateField('email', v)}
            placeholder="you@example.com"
            placeholderTextColor={Colors.textTertiary}
            keyboardType="email-address"
            autoCapitalize="none"
            returnKeyType="done"
            onSubmitEditing={handleSubmit}
          />
          {errors.email && <Text style={styles.fieldError}>{errors.email}</Text>}
        </View>

        {/* Submit error */}
        {submitError && <Text style={styles.submitError}>{submitError}</Text>}

        {/* Submit button */}
        <TouchableOpacity
          style={[styles.button, isSubmitDisabled && styles.buttonDisabled]}
          onPress={handleSubmit}
          disabled={isSubmitDisabled}
          activeOpacity={0.8}
        >
          <Text style={styles.buttonText}>
            {isLoading ? 'Saving...' : 'Save'}
          </Text>
        </TouchableOpacity>
      </ScrollView>
    </KeyboardAvoidingView>
  );
}

// 4. Styles at the bottom
const styles = StyleSheet.create({
  keyboardAvoid: {
    flex: 1,
  },
  scrollContent: {
    padding: Spacing.lg,
    gap: Spacing.lg,
  },
  fieldGroup: {
    gap: Spacing.xs,
  },
  label: {
    fontSize: Typography.size.sm,
    fontWeight: Typography.weight.medium,
    color: Colors.textSecondary,
  },
  input: {
    backgroundColor: Colors.surfaceRaised,
    borderRadius: Radius.md,
    borderWidth: 1,
    borderColor: Colors.border,
    paddingHorizontal: Spacing.md,
    paddingVertical: Spacing.md,
    fontSize: Typography.size.md,
    color: Colors.textPrimary,
    minHeight: 48,
  },
  inputError: {
    borderColor: Colors.danger,
  },
  fieldError: {
    fontSize: Typography.size.xs,
    color: Colors.danger,
  },
  submitError: {
    fontSize: Typography.size.sm,
    color: Colors.danger,
    textAlign: 'center',
  },
  button: {
    backgroundColor: Colors.primary,
    borderRadius: Radius.md,
    paddingVertical: Spacing.md,
    alignItems: 'center',
    minHeight: 48,
    marginTop: Spacing.sm,
  },
  buttonDisabled: {
    opacity: 0.4,
  },
  buttonText: {
    fontSize: Typography.size.md,
    fontWeight: Typography.weight.semibold,
    color: Colors.textPrimary,
  },
});
```

---

## Searchable Dropdown Pattern

For selectors (not free-text input):

```typescript
import { useState } from 'react';
import { View, Text, TextInput, FlatList, TouchableOpacity, StyleSheet } from 'react-native';
import { Colors, Spacing, Typography, Radius } from '@/config/theme';

interface SearchableDropdownProps {
  options: string[];
  value: string | null;
  onSelect: (value: string) => void;
  placeholder: string;
  label: string;
}

export function SearchableDropdown({
  options,
  value,
  onSelect,
  placeholder,
  label,
}: SearchableDropdownProps) {
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);

  const filtered = options.filter(o =>
    o.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <View style={styles.container}>
      <Text style={styles.label}>{label}</Text>
      <TouchableOpacity
        style={styles.selector}
        onPress={() => setIsOpen(prev => !prev)}
        activeOpacity={0.7}
      >
        <Text style={value ? styles.valueText : styles.placeholder}>
          {value ?? placeholder}
        </Text>
      </TouchableOpacity>

      {isOpen && (
        <View style={styles.dropdown}>
          <TextInput
            style={styles.searchInput}
            value={query}
            onChangeText={setQuery}
            placeholder="Search..."
            placeholderTextColor={Colors.textTertiary}
            autoFocus
          />
          <FlatList
            data={filtered}
            keyExtractor={item => item}
            style={styles.list}
            renderItem={({ item }) => (
              <TouchableOpacity
                style={styles.option}
                onPress={() => {
                  onSelect(item);
                  setIsOpen(false);
                  setQuery('');
                }}
              >
                <Text style={styles.optionText}>{item}</Text>
              </TouchableOpacity>
            )}
          />
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { gap: Spacing.xs },
  label: {
    fontSize: Typography.size.sm,
    fontWeight: Typography.weight.medium,
    color: Colors.textSecondary,
  },
  selector: {
    backgroundColor: Colors.surfaceRaised,
    borderRadius: Radius.md,
    borderWidth: 1,
    borderColor: Colors.border,
    paddingHorizontal: Spacing.md,
    paddingVertical: Spacing.md,
    minHeight: 48,
    justifyContent: 'center',
  },
  valueText: {
    fontSize: Typography.size.md,
    color: Colors.textPrimary,
  },
  placeholder: {
    fontSize: Typography.size.md,
    color: Colors.textTertiary,
  },
  dropdown: {
    backgroundColor: Colors.surfaceRaised,
    borderRadius: Radius.md,
    borderWidth: 1,
    borderColor: Colors.border,
    maxHeight: 240,
  },
  searchInput: {
    padding: Spacing.md,
    borderBottomWidth: 1,
    borderBottomColor: Colors.border,
    fontSize: Typography.size.md,
    color: Colors.textPrimary,
  },
  list: { flexGrow: 0 },
  option: {
    padding: Spacing.md,
    borderBottomWidth: 1,
    borderBottomColor: Colors.border,
  },
  optionText: {
    fontSize: Typography.size.md,
    color: Colors.textPrimary,
  },
});
```

---

## Rules

1. **No external form libraries.** Plain `useState` only.
2. **Validation runs on submit**, not on every keystroke. Clear field error on change.
3. **`KeyboardAvoidingView` wraps every form** — `padding` on iOS, `height` on Android.
4. **`keyboardShouldPersistTaps="handled"`** on the `ScrollView` — prevents tap dismissal issues.
5. **`FormErrors` is `Partial<Record<keyof FormFields, string>>`** — only fields with errors appear.
6. **Submit error is separate from field errors** — network/server errors show below the form.
7. **Submit button disabled** when required fields are empty or while loading.
8. **`minHeight: 48`** on all inputs and buttons — accessibility touch target.
9. **`returnKeyType`** set appropriately: `"next"` for non-last fields, `"done"` for last field.
10. **Styles at the bottom** of the file, using theme tokens only.
