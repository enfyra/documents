# User Registration Example

This example demonstrates how to create a custom `/api/register` endpoint using Enfyra's Custom Handler system. The endpoint handles user registration with validation, password hashing, and proper response formatting.

## Overview

**Why Custom Handler?**
- Default `/api/user_definition` POST creates users but doesn't include registration-specific validation
- Need custom email/password validation rules
- Want to hash passwords before storage
- Need to check for duplicate emails
- Return custom success/error messages

## Step 1: Create the Route

1. Navigate to **Settings → Routes** → [📖 Routing Management Guide](../frontend/routing-management.md)
2. Click **"Create New Route"**
3. Configure:
   - **Route**: `/register`
   - **Method**: Select `POST`
   - **Handler**: Select `Custom` 
   - **Target Tables**: Add `user_definition` (so we can access via `$ctx.$repos.user_definition`)
4. Save the route

## Step 2: Create the Custom Handler

1. Navigate to **Settings → Handlers** → [📖 Custom Handlers Guide](../frontend/custom-handlers.md)
2. Click **"Create New Handler"**
3. Configure:
   - **Route**: Click relation picker → Select `/register` route
   - **Method**: Click relation picker → Select `POST`
   - **Logic**: Enter the JavaScript code below
4. Save the handler

### Handler Code:
```javascript
// POST /register Custom Handler
// Target Tables: user_definition

const { email, password, name } = $ctx.$body;

$ctx.$logs(`Registration attempt for email: ${email}`);

// 1. Validate required fields
if (!email || !password) {
  $ctx.$logs('Registration failed: Missing required fields');
  $ctx.$throw['400']('Email and password are required', {
    fields: {
      email: !email ? 'Email is required' : null,
      password: !password ? 'Password is required' : null
    }
  });
}

// 2. Validate email format
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test(email)) {
  $ctx.$logs(`Registration failed: Invalid email format: ${email}`);
  $ctx.$throw['400']('Invalid email format');
}

// 3. Validate password strength
if (password.length < 8) {
  $ctx.$logs('Registration failed: Password too short');
  $ctx.$throw['400']('Password must be at least 8 characters long');
}

// 4. Check for duplicate email
const existingUserResult = await $ctx.$repos.user_definition.find({
  where: { email: { _eq: email } },
  fields: 'id,email' // Only check existence, minimal data
});

if (existingUserResult.data.length > 0) {
  $ctx.$logs(`Registration failed: Email already exists: ${email}`);
  $ctx.$throw['409']('User', 'email', email);
}

$ctx.$logs('Validation passed, proceeding with user creation');

// 5. Hash password
const hashedPassword = await $ctx.$helpers.$bcrypt.hash(password);
$ctx.$logs('Password hashed successfully');

// 6. Create user (auto-calls .find() after insert to return full record)
const userResult = await $ctx.$repos.user_definition.create({
  email: email,
  password: hashedPassword, // Store hashed version
  name: name || null, // Optional field
  isActive: true,
  createdAt: new Date().toISOString()
});

// The create() method automatically calls .find() after insert
// so userResult.data[0] contains the full record with ID, timestamps, etc.
const newUser = userResult.data[0];

$ctx.$logs(`User created successfully with ID: ${newUser.id}`);

// 7. Return success response (never return password)
return {
  success: true,
  message: 'User registered successfully',
  user: {
    id: newUser.id,
    email: newUser.email,
    name: newUser.name,
    isActive: newUser.isActive,
    createdAt: newUser.createdAt
    // Password intentionally excluded
  },
  logs: $ctx.$share.$logs // Include execution logs for debugging
};
```

## Step 3: Install Nodemailer Package

1. Navigate to **Settings → Packages** → [📖 Package Management Guide](../frontend/package-management.md)
2. Click **"Install Package"**
3. Select **"Backend Package"** type
4. Search for `nodemailer`
5. Click on `nodemailer` from dropdown
6. Click **"Install"** to add the package

## Step 4: Create AfterHook for Welcome Email

1. Navigate to **Settings → Hooks** → [📖 Hooks System Guide](../frontend/hooks.md)
2. Click **"Create New Hook"**
3. Configure:
   - **Route**: Click relation picker → Select `/register` route
   - **Method**: Click relation picker → Select `POST`
   - **AfterHook**: Enter the welcome email code below
   - **Priority**: `10` (run after main handler)
4. Save the hook

### AfterHook Code:
```javascript
// Welcome Email AfterHook
// Only runs if registration was successful (no $ctx.$api.error)

if (!$ctx.$api.error && $ctx.$data.success && $ctx.$data.user) {
  $ctx.$logs('Sending welcome email notification');
  
  const nodemailer = $ctx.$pkgs.nodemailer;
  
  // Configure transporter (adjust with your SMTP settings)
  const transporter = nodemailer.createTransporter({
    host: 'smtp.gmail.com',
    port: 587,
    secure: false, // true for 465, false for other ports
    auth: {
      user: 'your-email@gmail.com',
      pass: 'your-app-password'
    }
  });

  // Email content
  const mailOptions = {
    from: '"Your App" <noreply@yourapp.com>',
    to: $ctx.$data.user.email,
    subject: '🎉 Welcome to Our Platform!',
    html: `
      <h2>Welcome ${$ctx.$data.user.name || 'there'}!</h2>
      <p>Thank you for registering with our platform.</p>
      
      <div style="background: #f5f5f5; padding: 20px; border-radius: 8px; margin: 20px 0;">
        <h3>Your Account Details:</h3>
        <p><strong>Email:</strong> ${$ctx.$data.user.email}</p>
        <p><strong>Account ID:</strong> ${$ctx.$data.user.id}</p>
        <p><strong>Registered:</strong> ${new Date($ctx.$data.user.createdAt).toLocaleDateString()}</p>
      </div>
      
      <p>You can now log in to access all features of our platform.</p>
      
      <p>Best regards,<br>
      The Team</p>
    `
  };

  try {
    // Send email
    const info = await transporter.sendMail(mailOptions);
    $ctx.$logs(`Welcome email sent successfully: ${info.messageId}`);
    
  } catch (error) {
    $ctx.$logs(`Failed to send welcome email: ${error.message}`);
    // Don't throw error - email failure shouldn't break registration
  }
} else {
  $ctx.$logs('Skipping welcome email - registration failed');
}
```

## Step 5: Test the Endpoint

### Via cURL:
```bash
# Successful registration
curl -X POST http://localhost:1105/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "securepassword123",
    "name": "John Doe"
  }'

# Missing fields
curl -X POST http://localhost:1105/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com"
  }'

# Duplicate email
curl -X POST http://localhost:1105/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@example.com",
    "password": "password123"
  }'
```

### Expected Responses:

**Success Response:**
```json
{
  "success": true,
  "message": "User registered successfully",
  "user": {
    "id": 123,
    "email": "john@example.com",
    "name": "John Doe",
    "isActive": true,
    "createdAt": "2024-01-15T10:30:00.000Z"
  },
  "logs": [
    "Registration attempt for email: john@example.com",
    "Validation passed, proceeding with user creation",
    "Password hashed successfully",
    "User created successfully with ID: 123",
    "Product created: John Doe"
  ]
}
```

**Error Response (Duplicate Email):**
```json
{
  "statusCode": 409,
  "message": "Conflict",
  "details": "User with email 'john@example.com' already exists",
  "logs": [
    "Registration attempt for email: john@example.com",
    "Registration failed: Email already exists: john@example.com"
  ]
}
```

## Step 4: Frontend Integration

### Using useApi() Composable:
```vue
<template>
  <div class="p-6">
    <UForm @submit="handleRegister">
      <UInput 
        v-model="form.email" 
        placeholder="Email"
        type="email"
        required
      />
      <UInput 
        v-model="form.password" 
        placeholder="Password"
        type="password"
        required
      />
      <UInput 
        v-model="form.name" 
        placeholder="Full Name"
      />
      <UButton 
        type="submit"
        :loading="loading"
        color="primary"
      >
        Register
      </UButton>
    </UForm>
  </div>
</template>

<script setup>
const toast = useToast();
const router = useRouter();

const form = reactive({
  email: '',
  password: '',
  name: ''
});

const loading = ref(false);

const handleRegister = async () => {
  loading.value = true;
  
  try {
    const { data, error } = await useApi('/register', {
      method: 'POST',
      body: form
    });

    if (error.value) {
      toast.add({
        title: 'Registration Failed',
        description: error.value.message,
        color: 'red'
      });
      return;
    }

    // Success
    toast.add({
      title: 'Registration Successful',
      description: `Welcome ${data.value.user.name}!`,
      color: 'green'
    });

    // Redirect to login or dashboard
    router.push('/login');

  } catch (err) {
    toast.toast.add({
      title: 'Unexpected Error',
      description: 'Please try again later',
      color: 'red'
    });
  } finally {
    loading.value = false;
  }
};
</script>
```

## Advanced Features

### Optional: Add PreHook for Additional Validation

If you want to add extra validation (like domain restrictions), you can create an additional PreHook:

1. Navigate to **Settings → Hooks** → [📖 Hooks System Guide](../frontend/hooks.md)
2. Click **"Create New Hook"**
3. Configure:
   - **Route**: Click relation picker → Select `/register` route
   - **Method**: Click relation picker → Select `POST`
   - **PreHook**: Enter the validation code below
   - **Priority**: `0` (run first, before handler)

```javascript
// PreHook validation runs before handler

// Email domain validation example
if ($ctx.$body.email && !$ctx.$body.email.endsWith('@company.com')) {
  $ctx.$logs('Registration blocked: Invalid email domain');
  $ctx.$throw['400']('Only company email addresses are allowed');
}

$ctx.$logs('PreHook validation passed');
```

## Security Considerations

### Password Security:
- ✅ Passwords are hashed using bcrypt before storage
- ✅ Never log or return password fields
- ✅ Minimum 8-character password requirement

### Validation:
- ✅ Email format validation
- ✅ Duplicate email checking
- ✅ Required field validation
- ✅ SQL injection prevention (via repository methods)

### Logging:
- ✅ Registration attempt logging
- ✅ Error logging with context
- ✅ Success confirmation logging
- ✅ Welcome email confirmation logging

## Troubleshooting

### Common Issues:

**Registration Failed: Missing Required Fields**
- Check that `email` and `password` are provided
- Verify Content-Type header is `application/json`

**Registration Failed: Email Already Exists**
- User with this email is already registered
- Suggest login instead or password reset

**Handler Not Found**
- Ensure route `/api/register` exists
- Ensure handler is linked to route
- Check handler is enabled

**Permission Denied**
- Ensure user has permission to create users
- Check route permissions in Settings

### Debug Checklist:
1. ✅ Route configured with `/api/register` path
2. ✅ Handler linked to route and POST method
3. ✅ Target Tables includes `user_definition`
4. ✅ Handler logic syntax is valid JavaScript
5. ✅ Test data includes valid email and password

This registration endpoint provides a complete, production-ready user registration system with proper validation, security, and error handling.
