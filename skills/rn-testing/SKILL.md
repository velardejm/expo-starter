---
name: rn-testing
description: Automated testing standard for React Native (Expo) apps. Use this skill when adding any test file. Covers what to test, what to skip, file naming, and the minimal setup needed. Keeps testing simple and high-value — no over-testing.
---

## The Rule

**Test logic, not implementation. Test domain functions and repositories — not UI rendering.**
If a test is harder to write than the code it tests, the code is probably too complex.

---

## What to Test (High Value)

| Target | Why |
|--------|-----|
| Domain mappers | Pure functions — easy to test, high risk if broken |
| Validation functions (from rn-forms) | Pure functions — fast, no mocks needed |
| Repository error handling | Catches silent Supabase failure modes |
| Offline queue logic | Complex enough to break silently |
| Business logic in domain hooks | State transitions, score calculation |

## What NOT to Test (Low Value / High Cost)

| Target | Why to skip |
|--------|-------------|
| Component rendering | Snapshot tests break on every UI change |
| Navigation flows | E2E tools handle this better |
| Supabase auth | Integration test, not unit test |
| Context providers | Test the hook, not the wrapper |
| StyleSheet values | Not behavior |

---

## Setup

```bash
# Already included with Expo — no extra install needed
npx expo install jest-expo @testing-library/react-native
```

```json
// package.json
"jest": {
  "preset": "jest-expo",
  "setupFilesAfterFramework": ["@testing-library/react-native/extend-expect"]
}
```

---

## File Naming Convention

```
src/domain/[entity]/[entity]Mappers.test.ts
src/domain/[entity]/[entity]Repository.test.ts
src/features/[feature]/model/use[Feature].test.ts
src/services/offlineQueue.test.ts
```

Tests live **next to the file they test** — not in a separate `__tests__` folder.

---

## Mapper Test Pattern (Most Common)

```typescript
// src/domain/build/buildMappers.test.ts

import { mapBuildFromDTO } from './buildMappers';

describe('mapBuildFromDTO', () => {
  const validDTO = {
    id: 'uuid-123',
    user_id: 'user-456',
    blade: 'Dran Sword',
    ratchet: '3-60',
    bit: 'Point',
    nickname: 'My Build',
    created_at: '2025-01-01T00:00:00Z',
  };

  it('maps a valid DTO to a Build', () => {
    const result = mapBuildFromDTO(validDTO);
    expect(result).toEqual({
      id: 'uuid-123',
      userId: 'user-456',
      blade: 'Dran Sword',
      ratchet: '3-60',
      bit: 'Point',
      nickname: 'My Build',
      createdAt: '2025-01-01T00:00:00Z',
    });
  });

  it('throws when id is missing', () => {
    expect(() => mapBuildFromDTO({ ...validDTO, id: undefined })).toThrow();
  });

  it('throws when given a non-object', () => {
    expect(() => mapBuildFromDTO(null)).toThrow();
    expect(() => mapBuildFromDTO('string')).toThrow();
  });

  it('handles null nickname', () => {
    const result = mapBuildFromDTO({ ...validDTO, nickname: null });
    expect(result.nickname).toBeNull();
  });
});
```

---

## Validation Function Test Pattern

```typescript
// src/features/auth/model/validate.test.ts

import { validate } from './validate';

describe('validate', () => {
  it('returns no errors for valid input', () => {
    const errors = validate({ name: 'JM', email: 'jm@test.com' });
    expect(Object.keys(errors)).toHaveLength(0);
  });

  it('requires name', () => {
    const errors = validate({ name: '', email: 'jm@test.com' });
    expect(errors.name).toBeDefined();
  });

  it('requires valid email format', () => {
    const errors = validate({ name: 'JM', email: 'notanemail' });
    expect(errors.email).toBeDefined();
  });
});
```

---

## Repository Test Pattern (with Supabase mock)

```typescript
// src/domain/build/buildRepository.test.ts

// Mock the supabase service — never hit a real DB in unit tests
jest.mock('@/services/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn().mockReturnThis(),
      eq: jest.fn().mockReturnThis(),
      order: jest.fn().mockReturnThis(),
      maybeSingle: jest.fn().mockResolvedValue({
        data: {
          id: 'uuid-123',
          user_id: 'user-456',
          blade: 'Dran Sword',
          ratchet: '3-60',
          bit: 'Point',
          nickname: null,
          created_at: '2025-01-01T00:00:00Z',
        },
        error: null,
      }),
    })),
  },
}));

import { getBuildById } from './buildRepository';

describe('getBuildById', () => {
  it('returns a mapped Build when found', async () => {
    const result = await getBuildById('uuid-123');
    expect(result?.id).toBe('uuid-123');
    expect(result?.blade).toBe('Dran Sword');
  });
});
```

---

## Running Tests

```bash
# Run all tests
npx jest

# Run tests for a specific file
npx jest src/domain/build/buildMappers.test.ts

# Watch mode during development
npx jest --watch

# Coverage report
npx jest --coverage
```

---

## Rules

1. **Test files live next to the source file** — not in a separate folder.
2. **Only test pure functions and repository error handling** in unit tests.
3. **Mock `@/services/supabase`** in repository tests — never hit a real DB.
4. **One `describe` block per file, one `it` per behavior.**
5. **Test names describe behavior**, not implementation: `'returns null when not found'` not `'calls maybeSingle'`.
6. **No snapshot tests** — they break on every UI change and add no value.
7. **Tests must pass before a phase is marked complete** — add to completion criteria when tests exist.
8. **Don't test TypeScript types** — the compiler already does that.
9. **Keep tests fast** — if a test takes more than 1 second, something is wrong.
10. **Write the test when writing the function** — not as a separate task later.
