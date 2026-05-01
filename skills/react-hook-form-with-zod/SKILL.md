---
name: rhf-zod
description: Rules and patterns for building React forms with React Hook Form (RHF) and Zod validation. Use this skill whenever the user is creating, editing, or refactoring any React form — including login forms, registration flows, multi-step wizards, dynamic field arrays, or any input component wired to RHF. Also trigger when the user mentions `useForm`, `Controller`, `zodResolver`, `z.object`, schema validation, form state, `useFieldArray`, or `FormProvider`. Trigger even if they just ask "how do I validate this field" or "how do I handle server errors in a form" — this skill covers it all.
---

# RHF + Zod Rules

## Core Laws

- Every form uses RHF — no `useState` for field values
- One form per file, one schema per form, all validation via Zod in `.schema.ts`
- `Controller` pattern only — never `register` on custom inputs
- Never write manual types — always `z.infer<typeof schema>`
- Always: `noValidate`, `defaultValues`, `handleSubmit`

## File Structure

```
src/features/<feature>/
  schemas/login.schema.ts       # Zod schema + z.infer export
  forms/LoginForm.tsx           # One form component
  services/auth.service.ts      # API/mutation logic

src/components/inputs/
  ControlledInput.tsx           # Reusable controlled wrappers (ControlledX.tsx)
```

## Schema Pattern

```ts
// login.schema.ts
import { z } from "zod";
export const loginSchema = z.object({
  email: z.string().min(1, "Required").email("Enter a valid email"),
  password: z.string().min(1, "Required").min(8, "Min 8 characters"),
});
export type LoginFormValues = z.infer<typeof loginSchema>;
```

- `.min(1, "...")` not `.nonempty()` — user-friendly messages on every validator
- Cross-field: `.refine()` / `.superRefine()` at schema root
- Shared fields: extract base schema → `.extend()` / `.merge()`
- Coercion: `.transform()` (trim, parse numbers)

```ts
// Cross-field example
export const registerSchema = z
  .object({
    password: z.string().min(8, "Min 8 characters"),
    confirmPassword: z.string().min(1, "Required"),
  })
  .refine((d) => d.password === d.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  });
```

## Form Component Pattern

```tsx
const LoginForm = ({ onSubmit }) => {
  const {
    control,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: "", password: "" },
  });
  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <ControlledInput
        name="email"
        control={control}
        label="Email"
        error={errors.email}
      />
      <ControlledInput
        name="password"
        control={control}
        label="Password"
        type="password"
        error={errors.password}
      />
      {errors.root && <div role="alert">{errors.root.message}</div>}
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Signing in..." : "Sign In"}
      </button>
    </form>
  );
};
```

## ControlledInput Wrapper

```tsx
const ControlledInput = ({
  name,
  control,
  label,
  type = "text",
  placeholder,
  error,
  ...rest
}) => (
  <div>
    {label && <label htmlFor={name}>{label}</label>}
    <Controller
      name={name}
      control={control}
      render={({ field }) => (
        <input
          {...field}
          id={name}
          type={type}
          placeholder={placeholder}
          aria-invalid={!!error}
          aria-describedby={error ? `${name}-error` : undefined}
          {...rest}
        />
      )}
    />
    {error && (
      <span id={`${name}-error`} role="alert">
        {error.message}
      </span>
    )}
  </div>
);
```

Non-standard APIs — map manually:

```tsx
<Controller
  name="category"
  control={control}
  render={({ field }) => (
    <CustomSelect
      value={field.value}
      onSelect={(v) => field.onChange(v)}
      onBlur={field.onBlur}
    />
  )}
/>
```

## Error Handling

- Field-level: `errors.<field>.message` next to input
- Server errors: `setError("root", { message: "..." })` → `errors.root?.message`
- A11y: `aria-invalid`, `aria-describedby`, `role="alert"`, `<label htmlFor>` + `id` on every input
- `shouldFocusError: true` (default) — auto-focuses first errored field

## Submission

```tsx
const onSubmit = (data) => {
  mutate(data, {
    onError: () => form.setError("root", { message: "Login failed" }),
  });
};
```

- Thin submit handlers — delegate to service/mutation layer
- Disable button during `isSubmitting`, `reset()` after success when needed
- `onSubmit` prop for modal/wizard forms; direct service call for self-contained forms

## Multi-Step Forms

- Each step = own component + own schema file
- Aggregate step data via context/zustand/parent
- Validate each step independently on "Next"; final submit against combined schema

## Dynamic / Array Fields

```tsx
const { fields, append, remove } = useFieldArray({ control, name: "items" });
// key={field.id} — never key on index
{
  fields.map((field, i) => (
    <div key={field.id}>
      <ControlledInput
        name={`items.${i}.description`}
        control={control}
        label="Desc"
      />
      <button type="button" onClick={() => remove(i)}>
        Remove
      </button>
    </div>
  ));
}
<button type="button" onClick={() => append({ description: "", qty: 1 })}>
  Add
</button>;
```

Schema: `z.array(z.object({ description: z.string(), qty: z.number() }))`

## FormProvider (≥3 nesting levels only)

```tsx
// Parent
<FormProvider {...methods}>
  <form onSubmit={methods.handleSubmit(onSubmit)} noValidate>
    ...
  </form>
</FormProvider>;
// Deep child
const {
  control,
  formState: { errors },
} = useFormContext();
```

## Performance

- `useWatch({ name: "field" })` — never `watch()` with no args
- Extract heavy input groups into sub-components (Controller scopes re-renders)
- `useMemo` for expensive derived values

## Anti-Patterns

| ❌ Don't                     | ✅ Do                                                |
| ---------------------------- | ---------------------------------------------------- |
| `useState` for fields        | `useForm`                                            |
| `register()` on custom input | `Controller`                                         |
| Inline schema in component   | `.schema.ts` file                                    |
| Manual `type FormValues`     | `z.infer<typeof schema>`                             |
| Validation in `onSubmit`     | Zod schema                                           |
| Multiple forms in one file   | One form per file                                    |
| `watch()` no args            | `useWatch({ name })`                                 |
| Missing `defaultValues`      | Always provide them                                  |
| Missing `noValidate`         | Always add it                                        |
| No `aria-*` on errors        | `aria-invalid` + `aria-describedby` + `role="alert"` |
| No submit guard              | Disable during `isSubmitting`                        |

## Dependencies

```bash
npm install react-hook-form zod @hookform/resolvers
# react-hook-form@^7.50  zod@^3.22  @hookform/resolvers@^3.3
```

## New Form Checklist

- [ ] `schemas/form-name.schema.ts` — Zod schema + `z.infer` type
- [ ] `forms/FormName.tsx` — `useForm` + `zodResolver` + `defaultValues`
- [ ] `Controller` wrappers for all inputs
- [ ] `noValidate`, `aria-*` errors, `isSubmitting` guard
- [ ] Server errors via `setError("root")`
- [ ] Thin submit → service/mutation layer
