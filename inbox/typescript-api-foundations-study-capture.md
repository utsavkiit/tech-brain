# TypeScript API Foundations — Study Capture

**In brief:** TypeScript types describe what the compiler may assume; they do not validate JSON at runtime. Model stable domain objects with interfaces or object types, model alternative outcomes with discriminated unions, and convert untrusted `unknown` input into trusted domain values through explicit validation.

## Object shapes and outcomes

An `interface` describes one object shape. It resembles the field definition of a Scala case class, but it does not generate a constructor, value equality, `copy`, pattern matching, or any runtime representation.

```ts
interface Receipt {
  readonly retailer: string;
  readonly items: readonly ReceiptItem[];
  readonly totalInCents: number;
}
```

A `type` is a general alias, not necessarily a union. The `|` operator creates a union. A discriminated union represents mutually exclusive outcomes while preventing impossible field combinations:

```ts
type ReceiptResult =
  | { status: "accepted"; receiptId: string }
  | { status: "duplicate"; receiptId: string }
  | { status: "invalid"; errors: string[] };
```

Switching on `status` narrows the value to the matching member. This is analogous to a Scala sealed hierarchy. `ReceiptResult` is an internal service outcome; the HTTP route can translate it into `201`, `409`, or `400` and choose the public JSON representation.

## Types and runtime values

A JavaScript object is a runtime collection of named properties and values. An object literal creates one:

```ts
const result = { status: "found", receipt };
```

A TypeScript declaration such as `type FindReceiptResult = ...` only describes which runtime values are valid. The declaration disappears during compilation, so it cannot be called like a Scala case-class constructor. The object is created with `{ ... }`; the type checks its shape.

```ts
type FindReceiptResult =
  | { status: "found"; receipt: Receipt }
  | { status: "not_found" };

const result: FindReceiptResult = { status: "found", receipt };
```

The caller accesses `result.status`, not `FindReceiptResult.status`. Narrowing on `result.status` proves whether `result.receipt` exists.

## Runtime validation

An HTTP body should initially be `unknown`, because a TypeScript annotation cannot prove that incoming JSON has the expected shape. Runtime checks establish each property before constructing a trusted `Receipt`; `any` would bypass those checks.

```ts
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}
```

The type predicate tells the compiler what a successful runtime check established. Nested arrays and their elements still need their own validation.

At runtime, `isRecord` returns only a boolean. Its `value is Record<string, unknown>` annotation additionally tells the compiler that a `true` result proves `value` is a non-null, non-array object. The property values remain unknown and require checks such as `typeof`, `Number.isInteger`, and `Array.isArray`.

Validation should produce a newly constructed domain object rather than return the caller's original object. Construction drops unknown fields, normalizes strings, and avoids retaining caller-owned mutable structures. Short-circuiting also makes checks safe: in `typeof value !== "string" || value.trim() === ""`, `.trim()` runs only after the first condition establishes a string.

## In-memory repositories and asynchronous contracts

`Record<string, V>` is a typed ordinary object with string keys. `Map<K, V>` is a key-value collection with explicit `set` and `get` operations and supports key types other than strings. `Map.get` returns `V | undefined` because a key may be absent.

```ts
class InMemoryReceiptRepository implements ReceiptRepository {
  private readonly receipts = new Map<string, Receipt>();

  async save(receipt: Receipt): Promise<string> {
    const id = crypto.randomUUID();
    this.receipts.set(id, receipt);
    return id;
  }

  async findById(id: string): Promise<Receipt | undefined> {
    return this.receipts.get(id);
  }
}
```

`Promise<T>` means an asynchronous operation eventually produces `T`; `await` extracts that eventual value inside an asynchronous function. Keeping even an in-memory repository behind a promise-returning interface allows a database-backed implementation later without changing callers. `async` wraps a directly returned value in a resolved promise.

## Absence, defaults, and failures

An optional property such as `note?: string` is effectively `string | undefined`. Optional chaining returns `undefined` instead of accessing a property on a missing value, and nullish coalescing supplies a fallback only for `null` or `undefined`:

```ts
const message = receipt?.note ?? "No note";
```

Unlike `||`, `??` preserves intentional values such as `""`, `0`, and `false`.

A repository returning `undefined` means the lookup completed but found no entity. A repository throwing means the lookup itself failed, perhaps because storage is unavailable. Converting every thrown error into `not_found` hides infrastructure failures and produces incorrect HTTP behavior.

## Exhaustive result handling

A `switch` on a discriminated union narrows the value in each case. An `assertNever` default makes the compiler report newly added outcomes that the caller has not handled:

```ts
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${JSON.stringify(value)}`);
}

switch (result.status) {
  case "found":
    return { statusCode: 200, body: result.receipt };
  case "not_found":
    return { statusCode: 404, body: { error: "Receipt not found" } };
  default:
    return assertNever(result);
}
```

`never` represents a value that should be impossible at that point. Adding a new union member without a matching case makes the default branch receive a non-`never` value and produces a compile error.

## Money representation

Store ordinary monetary amounts as integer minor units to avoid binary floating-point surprises such as `0.1 + 0.2 !== 0.3`.

```ts
interface Money {
  readonly amountInMinorUnits: number;
  readonly currency: "USD" | "CAD" | "JPY";
}
```

Currency answers which monetary system applies; minor units answer how the amount is encoded. `499 USD` minor units means $4.99, while `499 JPY` minor units means ¥499. Very large amounts may require `bigint` or a decimal library, and currency-specific exponent rules must be explicit.

## Tradeoffs and boundaries

`readonly` prevents reassignment through the typed reference, but it is not runtime immutability. Marking both an array property and the array itself readonly prevents replacing the property and calling mutating array methods through that reference. TypeScript types still disappear at runtime, so boundary validation remains necessary.

## Connections

- Discriminated unions connect domain modeling to exhaustive control flow and precise HTTP error mapping.
- Runtime validation is the bridge between untrusted transport data and compiler-checked domain logic.
- Promise-returning repository interfaces isolate callers from whether storage is in memory or performs asynchronous I/O.
- Explicit absence and thrown failures require different service and HTTP outcomes.
- Exhaustive switching turns domain-result changes into compiler-guided caller updates.

## Sources

- Personal study sessions, 2026-08-01 through 2026-08-02.
- ECMAScript/JavaScript number behavior discussed through the standard `0.1 + 0.2` floating-point example.
