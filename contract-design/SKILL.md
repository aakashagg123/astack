---
name: contract-design
description: >
  Contract-first API and interface design. Use this skill whenever the user is creating or
  changing an HTTP/RPC API, a public library interface, an SDK surface, a webhook, or any
  boundary that other code or other people will build against. Also trigger proactively when
  an implementation task involves a new endpoint or a breaking signature change — the contract
  gets designed and reviewed before the implementation starts, because an interface is the one
  part of a system you can't refactor freely once someone depends on it.
---

# Contract Design

You are designing a contract before writing an implementation. Implementations are cheap to
change; published interfaces are nearly permanent. Every hour spent on the contract saves ten
on migrations.

---

## Step 1 — Define the Contract First

Write the interface down before any implementation code:
- For HTTP APIs: the resource model, endpoints, request/response shapes (an OpenAPI sketch or
  equivalent table is enough).
- For libraries/SDKs: the public signatures, types, and errors — as they'd appear in docs.
- Review the contract against real call sites: write the *consumer's* code first, as it would
  ideally look. Awkward-to-call reveals bad design faster than any checklist.
- Check the codebase for existing conventions (naming, envelope shape, auth pattern, error
  format) and match them. Consistency inside a system beats external best practice.

---

## Step 2 — Design Checklist

Work through each; skip none silently:

**Shape**
- Nouns for resources, consistent casing, no leaking of internal storage names.
- Every collection endpoint paginated from day one (cursor-based unless there's a reason not
  to). Retrofitting pagination is a breaking change.
- Field types precise: timestamps in ISO 8601/UTC, money as integer minor units or decimal
  strings — never floats.

**Error semantics**
- Errors are part of the contract. Define the error shape once (machine-readable code,
  human-readable message, correlation id) and use it everywhere.
- Correct status/error categories: caller mistakes (4xx) vs. system faults (5xx). Never
  return success envelopes containing failures.
- Error messages must say what to *do*, not just what went wrong — and never leak internals
  (stack traces, queries, infrastructure names).

**Safety & idempotency**
- Reads are side-effect free. Unsafe operations that clients may retry (payments, creation)
  accept an idempotency key.
- Validate all inputs at the boundary; reject unknown-but-dangerous input loudly
  (pair with `/secure-coding-practices` for auth on every endpoint).

**Evolution**
- Additive changes (new optional fields) are safe; removals, renames, type changes, and
  semantic changes are breaking. Design so growth is additive.
- Pick the versioning story now (URL version, header, or "additive-only, no versions") —
  and state the deprecation path before shipping v1.
- Consumers must ignore unknown response fields; document that expectation.

---

## Step 3 — Specify Behaviour, Not Just Shape

For each operation, one line each on: authorization rule, validation rules, side effects,
failure modes, and rate/size limits. This becomes the acceptance criteria the implementation
is tested against (feed it to `/spec-creator` or `/red-green-tdd`).

---

## Step 4 — Then Implement

- Implement to the contract; when implementation pressure suggests changing the contract,
  change the *document* first and re-run the consumer-code sanity check.
- Contract tests: at minimum, tests that pin the response shapes and error codes, so an
  accidental breaking change fails CI instead of a consumer.

---

## Anti-Patterns to Refuse

- Designing the API by whatever the ORM/model happens to return.
- "We'll add errors handling/pagination/versioning later" — those are the contract.
- Booleans that will become enums (`status: active|suspended|deleted` beats `is_active`).
- Chatty interfaces that force N calls for one screen — design for the consumer's unit of work.
- Breaking a published interface without a deprecation window and a migration note.

---

## Exit Criteria

1. Written contract (shapes + error model + behaviour lines) exists and reads well from the
   consumer's side.
2. Conventions match the surrounding system.
3. Evolution story stated (versioning/deprecation).
4. Contract tests exist; implementation matches the document, not the other way around.
