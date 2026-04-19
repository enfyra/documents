# Guards

Guards protect your routes with declarative rules — no code required. Configure them through the admin UI or API to block IPs, rate-limit requests, and control access.

## How Guards Work

Guards run automatically at two points in the request lifecycle:

```
1. Route Detection
2. Pre-Auth Guards          <-- IP blocking, rate limiting (before login)
3. JWT Authentication
4. Role/Permission Check
5. Post-Auth Guards         <-- User-specific rate limiting (after login)
6. Pre-Hooks  Handler  Post-Hooks
```

- **Pre-auth** guards run before login. Use for IP-based rules and global rate limiting.
- **Post-auth** guards run after login. Use for user-specific rules.

## Rule Types

| Rule Type | Description | Config |
|---|---|---|
| `ip_whitelist` | Only allow listed IPs (blocks everyone else) | `{"ips": ["1.2.3.4", "10.0.0.0/8"]}` |
| `ip_blacklist` | Block listed IPs (allows everyone else) | `{"ips": ["5.6.7.8"]}` |
| `rate_limit_by_ip` | Rate limit per client IP per route | `{"maxRequests": 100, "perSeconds": 60}` |
| `rate_limit_by_user` | Rate limit per user per route (post-auth only) | `{"maxRequests": 50, "perSeconds": 60}` |
| `rate_limit_by_route` | Rate limit per route (all users share the limit) | `{"maxRequests": 200, "perSeconds": 60}` |

IP rules support **CIDR notation**: `10.0.0.0/8`, `192.168.1.0/24`, `172.16.0.0/12`.

When a rate limit is hit, the response returns **429** with headers: `Retry-After`, `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.

## Guard Tree

Guards can be combined using **AND** / **OR** logic to build complex rules.

### AND (all rules must pass)

```
AND Guard
├── ip_whitelist: only office IPs
└── rate_limit_by_ip: max 100/min
```

Both conditions must pass. If either fails, the request is rejected.

### OR (at least one rule must pass)

```
OR Guard
├── ip_whitelist: allow internal IPs (pass immediately)
└── rate_limit_by_ip: max 100/min (checked only if IP not whitelisted)
```

Internal IPs bypass rate limiting. External IPs get rate-limited.

### Nesting

You can nest guards for more complex logic:

```
OR Guard (root)
├── ip_whitelist: 10.0.0.0/8
└── AND Guard (child)
    ├── rate_limit_by_ip: max 100/min
    └── rate_limit_by_user: max 50/min
```

Result: Internal IPs pass immediately. External IPs must pass both rate limits.

## Configuration

### Create a Guard

```
POST /api/guard_definition
{
  "name": "Login Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<route_id>" },
  "methods": [
    { "id": "<POST_method_id>" }
  ]
}
```

| Field | Values | Description |
|---|---|---|
| `position` | `pre_auth`, `post_auth` | When the guard runs |
| `combinator` | `and`, `or` | How rules are combined (default: `and`) |
| `isGlobal` | `true` / `false` | Apply to all routes |
| `route` | relation | Apply to a specific route (null if global) |
| `methods` | relation | Apply to specific HTTP methods (empty = all methods) |
| `priority` | number | Execution order (lower = first) |

### Add Rules to a Guard

```
POST /api/guard_rule_definition
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 5, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

To scope a rule to specific users only:

```
POST /api/guard_rule_definition
{
  "type": "rate_limit_by_user",
  "config": { "maxRequests": 30, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" },
  "users": [
    { "id": "<user_id_1>" },
    { "id": "<user_id_2>" }
  ]
}
```

When `users` is empty, the rule applies to everyone.

### Nest Guards

Set `parent` to create a child guard:

```
POST /api/guard_definition
{
  "name": "External Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "parent": { "id": "<parent_guard_id>" },
  "isEnabled": true
}
```

## Common Patterns

### Block Bad IPs from All Routes

```
POST /api/guard_definition
{
  "name": "Global IP Blacklist",
  "position": "pre_auth",
  "combinator": "and",
  "isGlobal": true,
  "isEnabled": true
}

POST /api/guard_rule_definition
{
  "type": "ip_blacklist",
  "config": { "ips": ["1.2.3.4", "5.6.7.8"] },
  "guard": { "id": "<guard_id>" }
}
```

### Rate Limit Login Attempts

5 requests per minute per IP, before authentication:

```
POST /api/guard_definition
{
  "name": "Login Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<login_route_id>" },
  "methods": [{ "id": "<POST_method_id>" }]
}

POST /api/guard_rule_definition
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 5, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

### Office IP Only + Rate Limit for Admin Routes

```
POST /api/guard_definition
{
  "name": "Admin Access Control",
  "position": "post_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<admin_route_id>" }
}

POST /api/guard_rule_definition
{
  "type": "ip_whitelist",
  "config": { "ips": ["10.0.0.0/8", "203.0.113.0/24"] },
  "guard": { "id": "<guard_id>" }
}

POST /api/guard_rule_definition
{
  "type": "rate_limit_by_user",
  "config": { "maxRequests": 100, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

### Whitelist OR Rate Limit (Internal IPs bypass limits)

```
POST /api/guard_definition
{
  "name": "Internal or Rate Limited",
  "position": "pre_auth",
  "combinator": "or",
  "isGlobal": true,
  "isEnabled": true
}

POST /api/guard_rule_definition
{
  "type": "ip_whitelist",
  "config": { "ips": ["10.0.0.0/8"] },
  "guard": { "id": "<guard_id>" }
}

// Create child guard for external traffic
POST /api/guard_definition
{
  "name": "External Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "parent": { "id": "<root_guard_id>" },
  "isEnabled": true
}

POST /api/guard_rule_definition
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 100, "perSeconds": 60 },
  "guard": { "id": "<child_guard_id>" }
}
```

### Global API Rate Limit

```
POST /api/guard_definition
{
  "name": "Global API Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "isGlobal": true,
  "isEnabled": true
}

POST /api/guard_rule_definition
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 200, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

### User-Specific Stricter Limits

Apply stricter limits to specific users:

```
POST /api/guard_definition
{
  "name": "Heavy User Limit",
  "position": "post_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<api_route_id>" }
}

POST /api/guard_rule_definition
{
  "type": "rate_limit_by_user",
  "config": { "maxRequests": 30, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" },
  "users": [{ "id": "<user_id_1>" }, { "id": "<user_id_2>" }]
}
```

## Guards vs. preHook Rate Limiting

| Aspect | Guards | preHook (`$helpers.$rateLimit`) |
|---|---|---|
| Setup | Admin UI / API (no code) | Custom script |
| Runs before auth? | Yes (pre_auth) | No |
| Tree logic (AND/OR) | Built-in | Manual scripting |
| Best for | Standard IP/rate protection | Custom dynamic logic |

Use **guards** for standard, config-driven protection. Use **preHook rate limiting** when you need custom logic (conditional limits, dynamic keys, etc.).

## Notes

- `rate_limit_by_user` is only available in `post_auth` position (needs user context)
- Rules within a guard are evaluated cheapest first (IP checks before rate limit counters)
- Guards apply per instance — changes sync across instances via Redis Pub/Sub
- `priority` controls execution order — lower values run first

## Next Steps

- See [API Lifecycle](./api-lifecycle.md) for the full request flow
- See [Hooks and Handlers](./hooks-handlers/prehooks.md) for code-driven protection
- See [Context Reference - Rate Limiting](./context-reference/helpers-cache.md#rate-limiting) for preHook rate limiting
