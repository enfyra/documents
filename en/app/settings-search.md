---
slug: app/settings-search
---

# Find Settings records quickly

Use the table's Search field to find records without opening the advanced filter builder. Each Settings quick search targets exactly one field.

| Settings page | Search field |
|---|---|
| Users | `email` |
| Roles | `name` |
| Routes | `path` |
| Guards | `name` |
| Flows | `name` |
| Extensions | `name` |
| WebSockets | `path` |
| Helpers | `name` |
| API Tester | `path` |
| OAuth Accounts | `user.email` |
| Server Logs | Existing exact `correlationId` lookup |

An extension's associated menu path is not its own path, so Extensions searches its name. Methods, Auth Headers, OAuth Config and Menus have no quick-search field.

## Submit or clear a search

1. Type the field value. Typing changes the draft only; it does not send a request.
2. Press Enter or click **Search** to submit. Leading/trailing whitespace is removed, the list returns to page one, and the committed text appears in `?search=`.
3. Clear the text and submit to remove the search. Submitting the same search on page one refreshes the list.

Quick search uses the supported substring operator `_contains`, not `_icontains`. OAuth Accounts uses a nested `user.email` filter. JSON-looking text remains a literal field value; it is not interpreted as a filter object. Matching follows the server/database's `_contains` semantics, not a promised case-insensitive mode.

Existing advanced filters remain one complete group and combine with search through AND. Route System/Custom/All scope and Collection Routes state, Guard type, Helper status and API Tester scope remain in effect. Unrelated URL query values are preserved; browser Back/Forward restores the committed search and field draft. Filtered counts use `meta.filterCount`, including zero.

## Mobile layout

Search fills the first row of the table header below 768px and pushes the remaining controls below it. Desktop keeps the heading, search and quick filters on the left, with custom actions and Columns on the right. See [DataTable](./data-table.md) for extension toolbar and footer composition.
