# Profile Owner Scope

This example lets a logged-in user manage their own profile record.

## Tables

Create `user_profile`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `user` | one-to-one relation to `enfyra_user` | Required |
| `displayName` | string | Public display name |
| `bio` | text | Optional |
| `avatar` | many-to-one relation to `enfyra_file` | Optional |

Add a unique constraint on `user`.

## Read The Current Profile

Add a `GET /user_profile` pre-hook.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

if (!@USER?.id) {
  @THROW401();
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { user: { id: { _eq: @USER.id } } }
  ]
};
```

```bash
curl "$ENFYRA_API_URL/user_profile?limit=1&fields=id,displayName,bio,avatar.id,avatar.url" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

## Create Or Update Current Profile

Create a custom `POST /profile/me` handler.

```javascript
if (!@USER?.id) {
  @THROW401();
}

const existing = await #user_profile.find({
  filter: { user: { id: { _eq: @USER.id } } },
  fields: 'id',
  limit: 1
});

const data = {
  displayName: @BODY.displayName,
  bio: @BODY.bio,
  user: { id: @USER.id }
};

if (@BODY.avatar) {
  data.avatar = @BODY.avatar;
}

if (existing.data?.[0]) {
  const updated = await #user_profile.update({
    id: existing.data[0].id,
    data
  });
  return updated.data?.[0] || null;
}

const created = await #user_profile.create({ data });
return created.data?.[0] || null;
```

## Protect Direct Writes

For direct `POST /user_profile` and `PATCH /user_profile/:id`, add hooks that force the relation to the current user and scope updates to the owner. Root admins can still manage records from the admin console.
