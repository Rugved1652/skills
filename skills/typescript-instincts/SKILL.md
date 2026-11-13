---
name: typescript-instincts
description: Generate, review, and refactor readable TypeScript contracts and reusable types in Fastify, Next.js, and standalone TypeScript projects. Use for shared type organization, API request/response naming, generics, schema inference, conditional types, and strict type safety; not unrelated application architecture.
---

# TypeScript Instincts

Make types easy to find, reuse, and understand. Prefer one authoritative definition per concept, explicit API boundaries, and inference within implementations. Do not add clever abstractions merely to demonstrate TypeScript features.

## Inspect before declaring

1. Read repository instructions, `tsconfig`, package dependencies, import aliases, existing schemas/generated contracts, and neighboring modules.
2. Search for the concept and its existing names before creating a type. Inspect shared types, API contracts, schemas, and library exports. Reuse an existing definition only when its meaning and ownership match, not merely its fields.
3. Choose the source of truth: generated contract, runtime schema, existing domain type, or a new declaration. Derive from that source instead of maintaining parallel definitions.
4. Choose the narrowest appropriate owner using the placement rules below. Add only the files the current task needs.
5. Implement the type and its consumers together. Check affected imports, serialization, and request/response behavior before finishing.

## Type ownership and folders

For a new Fastify or Next.js standalone application, use this layout under the source root (`src/` when present, otherwise the project root):

```text
src/
  Types/
    common/
      generic.types.ts       # APIResponse<T>, shared pagination contracts
      enums.types.ts         # shared runtime enums, e.g. StatusCodes
    api-types/
      auth.types.ts          # auth request/response contracts
      billing.types.ts       # billing request/response contracts
    billing.types.ts         # reusable billing domain types, when needed
```

- Use the exact `Types` casing for this default; imports must match on case-sensitive systems. Preserve an established `types/`, feature-owned, or workspace contract-package layout instead of creating a competing tree. Explicit user placement requirements take precedence.
- `common/` owns domain-independent contracts with real shared use. A response envelope is common; a billing payment method is billing-owned even if several modules consume it.
- `api-types/` contains one file per API module, such as `auth.types.ts` and `billing.types.ts`. Split oversized modules by capability only when needed, e.g. `billing/invoice.types.ts`.
- Reusable non-transport domain types belong in `Types/<module>.types.ts` under this default. Keep implementation-only types local to their owning feature/function; do not move component props, database rows, or service internals into the API folder.
- Keep runtime validation schemas in their existing schema location. Contract files may export schema-derived aliases; do not copy the schema shape by hand.
- Dependencies flow from API contracts to domain types and common types. `common/` must not import feature/API definitions. Avoid circular imports and all-application barrel files; use focused exports when a public boundary needs them.
- Use `import type` / `export type` for type-only dependencies. Enums and schemas used at runtime need value imports. Do not import server libraries into shared client contracts.
- Do not create placeholder folders or reorganize unrelated code to populate this tree.

## Naming contract

Use PascalCase for interfaces, aliases, enums, and type parameters. Use kebab-case module filenames with `.types.ts` for contract files. Use camelCase for ordinary values/functions and the repository's existing schema naming convention.

| Concept | Pattern | Example |
|---|---|---|
| Reusable domain shape or union | `<Domain><Concept>Types` | `BillingMethodTypes`, `BillingAccountTypes` |
| API request body | `<Operation>APIRequest` | `SignUpAPIRequest`, `CreateInvoiceAPIRequest` |
| API response envelope | `<Operation>APIResponse` | `SignUpAPIResponse`, `CreateInvoiceAPIResponse` |
| API response payload | `<Operation>APIResponseData` | `SignUpAPIResponseData` |
| Other request parts | `<Operation>APIParams`, `<Operation>APIQuery`, `<Operation>APIHeaders` | `GetInvoiceAPIParams`, `ListInvoicesAPIQuery` |
| Shared generic | Semantic name with a type parameter | `APIResponse<TData>`, `PaginatedData<TItem>` |
| Enum | Semantic PascalCase name | `StatusCodes`, `BillingMethod` |
| Component contract | `<Component>Props` | `BillingCardProps` |

Use `BillingMethodTypes`, not `billingMethodTypes`. Do not add `I`/`T` prefixes to domain/API declarations, combine `InterfaceType`, or suffix every generic/enum with `Types`. Generic parameters such as `TData` are a separate convention. Keep operation names consistent across request, response, schema, and consumers. Preserve established public names unless a rename is part of the task; update affected consumers when changing a contract.

Use an `interface` for a straightforward object contract that benefits from extension. Use a `type` for unions, mapped/conditional types, schema inference, and generic instantiations. Both follow the same naming rules. Prefer `type SignUpAPIResponse = APIResponse<SignUpAPIResponseData>` over an empty interface that only extends the envelope. Interfaces cannot extend a union.

## Shared API contract

For a new JSON API using this convention, define the envelope once in `common/generic.types.ts`:

```ts
export interface APIResponse<TData> {
  data: TData;
  message: string;
  code: number;
  status: boolean;
}

export interface PaginatedData<TItem> {
  items: TItem[];
  total: number;
  page: number;
  pageSize: number;
}
```

- `boolean` is the TypeScript keyword; do not use `bool` or wrapper types such as `Boolean`.
- Require the payload type: `APIResponse<UserTypes>`, `APIResponse<UserTypes[]>`, or `APIResponse<PaginatedData<UserTypes>>`. Do not default it to `any` or repeat the envelope in every module.
- For this default, `status` indicates success and `code` is the HTTP status code. Keep the actual HTTP status and envelope consistent. If an existing API uses an application-specific code or a different shape, preserve that contract instead of silently remapping it.
- Explicitly represent empty JSON payloads with `null`, e.g. `APIResponse<null>`. An HTTP 204 response has no JSON body; do not add an envelope to it.
- The interface above does not encode a relationship between `status`, `code`, and `data`. When consumers need reliable success/failure narrowing, define the shared contract as a discriminated union with literal `status: true` / `status: false` and an explicit error payload. Do not claim that checking a plain `boolean` narrows `TData`.
- Reuse the envelope for transport JSON only. Domain services and database functions need not return HTTP envelopes. Never wrap an already-enveloped payload in a second `APIResponse`.
- Match the runtime serializer/validator to the envelope; a generic type does not configure runtime schemas.

Use `common/enums.types.ts` for shared runtime status codes when the project does not already provide an equivalent:

```ts
export enum StatusCodes {
  OK = 200,
  CREATED = 201,
  BAD_REQUEST = 400,
  UNAUTHORIZED = 401,
  FORBIDDEN = 403,
  NOT_FOUND = 404,
  INTERNAL_SERVER_ERROR = 500,
}
```

This is a representative set; add members when needed. Reuse installed status constants instead of maintaining a second list. Enums emit runtime values; avoid exporting `const enum` across package boundaries. If compiler/tooling settings prohibit enum syntax, use the existing constants or an `as const` object. For domain choices without a runtime enum requirement, a string literal union or `as const` object may be simpler. Follow the project's existing choice consistently.

## Reuse, generics, and inference

- Infer local variables and obvious returns. Annotate exported contracts, public function boundaries where useful, and ambiguous empty collections. Do not annotate every intermediate expression.
- Introduce generics when they preserve a relationship between inputs and outputs or reuse a container across payloads. Use descriptive parameters such as `TData`, `TItem`, `TKey`; a short `T` is fine in a small obvious helper.
- Constrain parameters to operations the implementation needs: `TKey extends keyof TItem`, or `TItem extends { id: string }`. Do not use `extends any` or generics whose type appears only once without preserving a useful relationship.
- Prefer built-ins such as `Pick`, `Omit`, `Record`, `Extract`, `Exclude`, `NonNullable`, `ReturnType`, `Parameters`, and `Awaited` before inventing equivalents. Use `infer` for a real extraction not already expressed clearly by a built-in.
- Derive DTOs with `Pick`/`Omit` only when they intentionally evolve with the source contract. Explicit allowlists or separate DTOs are safer for public responses and writable inputs; ORM entities often contain secrets, internal columns, or unserializable values.
- Use the existing validation library's inference (`z.infer`, `z.input`, `z.output`, TypeBox `Static`, etc.) rather than duplicating schema fields. Distinguish raw input from validated/transformed output. Do not add a validation library just for type inference.
- Use `satisfies` to check a value's compatibility while retaining its inferred shape; use `as const` for deliberate literal/readonly inference. Neither validates external data or freezes an object at runtime.
- Avoid speculative `DeepPartial`, recursive conditional types, and generic repositories/helpers with many knobs. Prefer a small domain-specific type when it communicates the contract better.

For concrete module contracts, constrained generics, schema inference, and conditional `infer`, read [references/type-patterns.md](references/type-patterns.md) when implementing those patterns.

## Safety and framework boundaries

- Avoid explicit and implicit `any`. Receive unknown external values as `unknown`, then validate or narrow. A cast, `fetch<T>`-style helper, or `JSON.parse` assignment cannot prove payload validity.
- Avoid non-null assertions, double casts through `unknown`, and suppression comments as fixes. Use guards, validation, or corrected contracts. Document only unavoidable interop assertions with the invariant that justifies them.
- Model finite choices with literal unions/enums; use discriminated unions for mutually exclusive states. Prefer exhaustive switches with a `never` check when adding a state must force callers to handle it.
- Distinguish absent properties (`field?: T`) from explicitly empty values (`field: T | null`). Avoid `Partial<Entity>` for create/update requests when it permits forbidden fields or ignores required business inputs.
- Prefer `unknown` over `{}`/`Object` for arbitrary data. Use `Record<string, TValue>` only for actual dictionaries; fixed shapes deserve named properties.
- Treat errors as `unknown` and narrow before accessing `.message`. Readonly types describe mutation constraints, not deep runtime immutability.
- In Fastify, bind body, params, query, headers, and replies through the existing route generics or schema type provider. Pair compile-time contracts with runtime schemas; keep each response status/schema aligned with the serialized output. Do not cast `request.body` or `reply` to bypass mismatches. Keep framework augmentation in an included `.d.ts` file and implement the corresponding runtime decoration.
- In Next.js, keep shared DTOs safe for client imports and appropriate to their transport. JSON APIs use serialized shapes such as date strings; React server-to-client props follow the framework's serialization rules. Derive public DTOs separately from database records and server-only dependencies. Validate requests and third-party responses at trust boundaries.
- Respect installed TypeScript/framework versions and module-resolution settings. Prefer strict checking; for a new project enable `strict`. Consider `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` when compatible. Do not weaken compiler flags or change repository-wide configuration to make one change pass.

## Verify the result

- Check for duplicated envelopes/types, inconsistent symbol casing, wrong ownership, circular dependencies, unsafe casts, and accidental server imports in client contracts.
- Run the affected project's lint, typecheck, tests, and build where available; inspect results and fix introduced failures. Use existing scripts/configuration rather than running a standalone compiler command that ignores the project's settings.
- For nontrivial generic or conditional behavior, use the existing type-test setup or focused compile-time checks for both accepted and rejected inputs. Test runtime validation/serialization separately; passing typechecks does not prove those behaviors.
- Report changed contracts and any compatibility impact, validation performed, and checks that could not run. Keep changes within the requested scope.
