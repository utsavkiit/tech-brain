# TypeScript API Foundations

## In brief

TypeScript types constrain what the compiler may assume, but they disappear at runtime. A reliable API therefore validates untrusted input before constructing domain values, models alternative outcomes with discriminated unions, and keeps transport concerns separate from domain and storage contracts.

## From untrusted input to domain values

Treat incoming JSON as `unknown`, not as an asserted request type. Runtime checks must establish the shape of every required property, including nested array elements, before application code can trust it.

```ts
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}
```

The type predicate records what the boolean check proves to the compiler; it does not perform additional runtime work. Property values remain `unknown` until checked with operations such as `typeof`, `Array.isArray`, or `Number.isInteger`.

After validation, construct a new domain object instead of retaining the caller's object. This drops unknown fields, permits normalization, and avoids sharing caller-owned mutable structures. `readonly` restricts mutation through a typed reference but does not freeze the runtime object, so it cannot replace boundary validation or defensive construction.

## Model shapes and outcomes separately

An `interface` or object type describes a runtime object's shape; it does not create a constructor, value equality, or any other runtime representation. Values are still created with object literals:

```ts
interface Receipt {
  readonly retailer: string;
  readonly totalInCents: number;
}

const receipt: Receipt = { retailer: "Corner Shop", totalInCents: 499 };
```

Use a discriminated union when an operation has a finite set of expected outcomes. The shared literal field lets TypeScript narrow each branch and prevents impossible field combinations; unexpected operational failures can still throw.

```ts
type ReceiptResult =
  | { status: "accepted"; receiptId: string }
  | { status: "duplicate"; receiptId: string }
  | { status: "invalid"; errors: string[] };
```

This result belongs to the service layer. The HTTP route translates it into status codes and a public JSON representation, keeping transport policy out of the domain model.

An exhaustive `switch` can make changes to the union compiler-visible at every caller:

```ts
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${JSON.stringify(value)}`);
}

function toHttpResponse(result: ReceiptResult) {
  switch (result.status) {
    case "accepted":
      return {
        statusCode: 201,
        body: { receiptId: result.receiptId },
      };

    case "duplicate":
      return {
        statusCode: 409,
        body: { receiptId: result.receiptId },
      };

    case "invalid":
      return {
        statusCode: 400,
        body: { errors: result.errors },
      };

    default:
      return assertNever(result);
  }
}
```

Each `case` narrows `result` to the matching union member, so only that member's fields are available. After every member has been handled, the `default` branch can receive only `never`. Adding a new result variant without a matching case makes `assertNever(result)` fail to compile, identifying the incomplete caller.

## Repository contracts preserve meaning

A repository can expose `Promise`-returning methods even when its first implementation uses an in-memory `Map`. Callers then do not need to change when storage becomes asynchronous, at the cost of using asynchronous calling conventions for in-memory work.

```ts
class InMemoryReceiptRepository implements ReceiptRepository {
  private readonly receipts = new Map<string, Receipt>();

  async findById(id: string): Promise<Receipt | undefined> {
    return this.receipts.get(id);
  }
}
```

`undefined` means the lookup completed and found no value; a thrown error means the lookup itself failed. Collapsing both into “not found” hides infrastructure failures and leads to incorrect API responses. Likewise, an optional property is effectively a union with `undefined`. Use `??` for a fallback only on `null` or `undefined`; unlike `||`, it preserves intentional values such as `0`, `false`, and `""`.

## Money needs an explicit representation

Represent ordinary amounts as integer minor units plus a currency instead of binary floating-point major units:

```ts
interface Money {
  readonly amountInMinorUnits: number;
  readonly currency: "USD" | "CAD" | "JPY";
}
```

The currency identifies the monetary system; its exponent determines what the minor-unit integer means. For example, `499` is $4.99 in USD but ¥499 in JPY. Currency rules must therefore stay explicit. This representation does not define rounding or currency conversion. Very large amounts may require `bigint`, while calculations needing fractional minor units or decimal rounding may require a decimal library.

## Sources

- Personal study sessions, 2026-08-01 through 2026-08-02, compiled from `inbox/typescript-api-foundations-study-capture.md`.
