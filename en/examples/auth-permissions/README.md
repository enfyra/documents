# Auth & Permissions

Use these examples when the main question is who can read or change a record.

Enfyra uses `enfyra_user` as the built-in user table. Relate application records directly to `enfyra_user` through real relations, then enforce ownership with route permissions, field permissions, pre-hooks, and handlers.

## Recipes

- [Profile Owner Scope](./profile-owner-scope.md) - Let users read and update their own profile fields.
- [Team Workspace RLS](./team-workspace-rls.md) - Share records inside a workspace while keeping other teams isolated.
- [Public Contact Form](./public-contact-form.md) - Accept anonymous submissions while blocking public reads.

## Rule Of Thumb

Use UI permissions for clarity, but always enforce the same rule on the server. A hidden button is not an access-control boundary.
