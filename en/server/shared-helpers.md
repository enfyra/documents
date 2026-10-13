---
slug: server/shared-helpers
---

# Reuse server operations with Shared Helpers

Shared Helpers let handlers, hooks, flow scripts and other dynamic server scripts call one saved operation by name without dispatching another route or starting a queue job. Use them for a reusable operation with explicit inputs and a clear result; keep request-specific routing and response orchestration in the caller.

## Prerequisites

Your Server must expose `enfyra_helper` and the shared helper runner, and your App must include **Settings → Helpers**. Verify the deployed release supports both before saving consumers. An initialized database requires a newer Server release whose bootstrap provisions the system table; an equal-version boot does not provision it. Do not reset initialization or downgrade the recorded version to bypass this check.

Helper authoring is privileged server-code authoring. The canonical `/enfyra_helper` CRUD route is private and has no default anonymous or non-root grant. Grant it only to trusted script authors.

## Create and test a helper

1. Open **Settings → Helpers** (`/settings/helpers`) and create a disabled draft.
2. Set a unique lowercase `name`, a description of inputs/results/side effects, and JavaScript or TypeScript as `scriptLanguage`.
3. Write a script body in `sourceCode`. Read explicit input through `@PARAMS`; return the result. There is no implicit `input` variable.
4. Use the **Test** tab with JSON params. Inspect the result before enabling and saving the helper.

Names contain 1–128 lowercase letters, digits or underscores. A digit may be first. `__proto__`, `prototype`, `constructor`, `run`, `then`, `catch` and `finally` are reserved. Enabled helpers must have executable source. Server generates `compiledCode`; never supply or edit it.

For a helper named `normalize_query`:

```typescript
const query: string = String(@PARAMS?.query ?? '').trim();
if (!query) @THROW400('query is required');
return { query };
```

The editor has **Editor**, **Usage** and **Test** tabs. Tests execute the current unsaved source with a five-second budget, without saving it. They can make real repository/network/storage changes and are not automatically rolled back. Nested named calls use saved activated helpers, not other unsaved drafts. Save, Reset, Delete and navigation retain dirty/pending guards.

## Call by name

```typescript
const result = await @HELPERS.run('normalize_query', {
  query: @BODY.query,
});
return result;
```

The equivalent API is `await $ctx.$helpers.run(name, params)`. A helper may call another helper. Params can be an object, array, primitive, null or undefined; they are not deep-cloned or made immutable. Always await the call, especially inside a transaction.

Each call has fresh local variables and data context. `@PARAMS` is the supplied argument; body, query, share and env are empty objects, and user is null. Caller request/API/result/flow data and lexical locals are not copied. Pass required information explicitly.

Built-in helpers, repositories, transactions, packages, cache, storage, sockets, logs and throws retain their existing capabilities. The helper shares the current task, JavaScript realm/globals, deadline and cancellation. This is a data-context boundary, **not a separate sandbox for hostile code**. It creates no child executor, HTTP request or route lifecycle replay.

## Preserve authorization and atomicity

A helper retains the caller's **host-owned repository authority**, even though its local user is null. Passing a user or role in params cannot grant authority. It does not replay another route's guards, validation, hooks or route-executed triggers. Put required authorization and business validation at the operation boundary, using secure repositories where appropriate.

```typescript
return await @TRANSACTION.run(async () => {
  await #orders.create({ data: { reference: @BODY.reference } });
  return await @HELPERS.run('reserve_inventory', {
    productId: @BODY.productId,
    quantity: @BODY.quantity,
  });
});
```

Awaited caller/helper repository calls join the active transaction and can read uncommitted writes. An error escaping the outer callback rolls back their database mutations together. A nested transaction joins the scope; it is not a savepoint or separate commit. Swallowing an error does not guarantee rollback.

Network, cache, socket, storage and queued-trigger effects are not undone by database rollback. Keep external work outside database locks when atomicity does not require it.

## Runtime updates and limits

Helpers are compiled and activated after metadata, before other script caches. Calls use the activated snapshot rather than reading the database each time. One task retains one coherent snapshot; activated edits affect subsequent tasks.

- Missing or disabled helpers fail explicitly. Ordinary errors name the helper; structured throws retain their status/carrier. No helper retry is added.
- The enclosing task's CPU, I/O, deadline and cancellation bounds remain authoritative.
- The catalogue supports at most 1,024 definitions; each task allows 1,024 calls and a call-chain depth of 32.
- All enabled helper executable source, caller source and wrapper overhead must fit the 4 MiB executable bound. A large catalogue can exceed it even when only one helper is called.
- Package references remain subject to existing package selection and authorization.

If the Helpers page or runner is missing, verify the deployed App/Server and system metadata first. If a call cannot find a helper, check its exact name and enabled state. Do not invent fallback route dispatch, retries or broader permissions.

See [Template Syntax](./template-syntax.md), [Helpers & Cache](./context-reference/helpers-cache.md) and [Repository Methods](./repository-methods/README.md) for the underlying script APIs.
