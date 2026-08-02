# TypeScript API Domain Modeling — Study Capture

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

## Runtime validation

An HTTP body should initially be `unknown`, because a TypeScript annotation cannot prove that incoming JSON has the expected shape. Runtime checks establish each property before constructing a trusted `Receipt`; `any` would bypass those checks.

```ts
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}
```

The type predicate tells the compiler what a successful runtime check established. Nested arrays and their elements still need their own validation.

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

## Sources

- Personal study session, 2026-08-01.
- ECMAScript/JavaScript number behavior discussed through the standard `0.1 + 0.2` floating-point example.
