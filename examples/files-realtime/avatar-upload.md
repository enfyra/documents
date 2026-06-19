# Avatar Upload

This example uploads a profile avatar and links it to `user_profile`.

## Upload Route

Create a custom `POST /profile/avatar` route that accepts multipart uploads. The handler stores the file and updates the current user's profile.

```javascript
if (!@USER?.id) {
  @THROW401();
}

if (!@UPLOADED_FILE) {
  @THROW400('file is required');
}

const uploaded = await @STORAGE.$upload({
  file: @UPLOADED_FILE,
  folder: 'avatars',
  isPublic: true
});

const profileResult = await #user_profile.find({
  filter: { user: { id: { _eq: @USER.id } } },
  fields: 'id',
  limit: 1
});

const data = {
  user: { id: @USER.id },
  avatar: { id: uploaded.id }
};

if (profileResult.data?.[0]) {
  const updated = await #user_profile.update({
    id: profileResult.data[0].id,
    data
  });
  return updated.data?.[0] || null;
}

const created = await #user_profile.create({ data });
return created.data?.[0] || null;
```

## Browser Request

```javascript
const form = new FormData();
form.append('file', fileInput.files[0]);

await fetch('/enfyra/profile/avatar', {
  method: 'POST',
  body: form,
  credentials: 'include'
});
```

## Public Profile Read

Expose only safe fields.

```text
GET /user_profile?fields=id,displayName,avatar.url
```
