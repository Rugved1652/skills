# Type Patterns

Use these examples when implementing the corresponding pattern. Adapt imports to the project's actual module resolution and preserve existing wire contracts.

## Module contracts reuse common and domain types

`Types/billing.types.ts`:

```ts
export type BillingMethodTypes = 'card' | 'bank_transfer';
```

`Types/api-types/billing.types.ts`:

```ts
import type { APIResponse } from '../common/generic.types';
import type { BillingMethodTypes } from '../billing.types';

export interface CreateInvoiceAPIRequest {
  accountId: string;
  method: BillingMethodTypes;
  amountInMinorUnits: number;
  currency: string;
}

export interface CreateInvoiceAPIResponseData {
  invoiceId: string;
  createdAt: string;
}

export type CreateInvoiceAPIResponse =
  APIResponse<CreateInvoiceAPIResponseData>;
```

`Types/api-types/auth.types.ts`:

```ts
import type { APIResponse } from '../common/generic.types';

export interface SignUpAPIRequest {
  email: string;
  password: string;
}

export interface SignUpAPIResponseData {
  userId: string;
  email: string;
}

export type SignUpAPIResponse = APIResponse<SignUpAPIResponseData>;
```

Response fields are an explicit public allowlist; do not derive this payload by omitting only `password` from a database user. Validate amount constraints, currency values, email, and password at runtime using the project's schema tooling. These interfaces alone enforce none of those value rules.

## Preserve relationships with a constrained generic

```ts
export const getProperty = <TItem, TKey extends keyof TItem>(
  item: TItem,
  key: TKey,
): TItem[TKey] => item[key];

const invoice = { id: 'inv_1', amountInMinorUnits: 2500 };
const amount = getProperty(invoice, 'amountInMinorUnits');
```

`amount` is inferred as `number`; invalid keys fail compilation. Do not replace this relationship with `key: string` and a cast. In `.tsx` files, disambiguate generic arrow syntax from JSX as needed (for example `<TItem,>` for a single unconstrained parameter).

## Prefer built-ins; use infer for targeted extraction

```ts
import type { APIResponse } from '../common/generic.types';

export type APIResponseData<TResponse extends APIResponse<unknown>> =
  TResponse extends APIResponse<infer TData> ? TData : never;

export type AsyncResult<TFunction extends (...args: never[]) => unknown> =
  Awaited<ReturnType<TFunction>>;
```

The relative import above assumes this helper is in `Types/api-types/`; if it is shared by unrelated modules, place it in `common/generic.types.ts` and use `APIResponse` directly. `APIResponseData` is useful when extracting payloads from existing contracts; direct indexed access (`SignUpAPIResponse['data']`) is clearer for a single known response. `AsyncResult` illustrates composition of built-ins; usually inline `Awaited<ReturnType<typeof operation>>` instead of declaring another alias. The `never[]` constraint avoids an `any[]` constraint and is not a callable runtime signature.

Conditional types distribute over union inputs when the checked side is a naked type parameter. Use tuple wrapping (`[T] extends [U]`) only when checking the entire union is intended. Keep such utilities small and verify their behavior for union/nullable inputs actually used by the application.

## Infer from runtime validation

When Zod is already installed, a module may use:

```ts
import { z } from 'zod';

export const signUpSchema = z.object({
  email: z.string().email(),
  password: z.string().min(12),
});

export type SignUpAPIRequest = z.input<typeof signUpSchema>;
export type ValidatedSignUpTypes = z.output<typeof signUpSchema>;

export const parseSignUp = (value: unknown): ValidatedSignUpTypes =>
  signUpSchema.parse(value);
```

Place the schema and aliases according to existing ownership rather than duplicating the schema in a types file. Handle parse failures through the application's validation/error boundary. Password length here is illustrative, not an instruction to change existing policy. For schemas with transformations/defaults, raw input and parsed output may differ; `z.infer` represents the output. Use the equivalent inference mechanism for the existing Fastify type provider instead of converting schemas to a different library.

## Optional discriminated API envelope

If designing a new API that needs success/failure narrowing, use this as the **replacement** definition in `common/generic.types.ts`, not a second competing envelope:

```ts
export type APIResponse<TData> =
  | { data: TData; message: string; code: number; status: true }
  | { data: null; message: string; code: number; status: false };
```

This preserves the four envelope keys and narrows `data` when checking `status`. It does not constrain valid HTTP success/error codes; enforce that in response builders and runtime tests, or narrow `code` to the project's explicit status sets. Operation response aliases continue to work. Existing boolean-status contracts require a deliberate compatibility review before adopting the union, including any non-null error payloads.
