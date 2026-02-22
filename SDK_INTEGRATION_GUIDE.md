# Aham Auth SDK Integration Guide

## Overview

The Aham Auth SDK provides seamless integration with the brah.ma Cross-Domain SSO system. This guide covers installation, configuration, and usage patterns for web applications.

## Installation

### CDN Integration (Recommended)

```html
<script src="https://aham.brah.ma/aham-auth-sdk.js"></script>
<script>
  // Configure the SDK
  window.ahamConfig = {
    clientId: 'your_client_id',
    subdomain: 'your_subdomain',
    authDomain: 'https://aham.brah.ma',
    debug: true // Enable for development
  }
</script>
```

### Local Installation

1. Download the SDK file:
```bash
curl -o aham-auth-sdk.js https://aham.brah.ma/aham-auth-sdk.js
```

2. Include in your project:
```html
<script src="./aham-auth-sdk.js"></script>
```

## Basic Configuration

### Initialize the SDK

```javascript
const auth = new AhamAuth({
  clientId: 'your_client_id',           // Your application ID
  subdomain: 'your_subdomain',          // Your subdomain (e.g., 'app' for app.brah.ma)
  authDomain: 'https://aham.brah.ma',   // Auth server URL
  projectId: 'your_project_id',         // Optional: Project identifier
  debug: true,                          // Enable debug logging
  
  // Event callbacks
  onLogin: (user) => {
    console.log('User logged in:', user)
    // Update UI, redirect, etc.
  },
  
  onLogout: () => {
    console.log('User logged out')
    // Clear UI, redirect to home, etc.
  },
  
  onError: (error) => {
    console.error('Auth error:', error)
    // Show error message to user
  }
})
```

### Auto-Configuration

For automatic initialization, define global config:

```javascript
// Define before SDK loads
window.ahamConfig = {
  clientId: 'your_client_id',
  subdomain: 'your_subdomain',
  onLogin: (user) => {
    document.getElementById('user-name').textContent = user.name
    document.getElementById('login-section').style.display = 'none'
    document.getElementById('user-section').style.display = 'block'
  },
  onLogout: () => {
    document.getElementById('login-section').style.display = 'block'
    document.getElementById('user-section').style.display = 'none'
  }
}
```

## Authentication Methods

### Standard Redirect Login

```javascript
// Redirect to auth server
auth.login({
  returnUrl: window.location.href,  // Where to return after login
  subdomain: 'app'                  // Target subdomain
})
```

### Popup Login (Better UX)

```javascript
// Login with popup window
auth.login({
  popup: true,
  returnUrl: window.location.href
}).then(user => {
  console.log('Login successful:', user)
}).catch(error => {
  console.error('Login failed:', error)
})
```

### Check Authentication Status

```javascript
// Check if user is logged in
if (auth.isAuthenticated()) {
  const user = auth.getUser()
  console.log('Current user:', user)
} else {
  console.log('User not authenticated')
}
```

### Session Management

```javascript
// Get current session
const session = auth.getSession()
if (session) {
  console.log('Token:', session.token)
  console.log('User:', session.user)
}

// Clear session
auth.logout()
```

## User Profile Management

### Get User Profile

```javascript
auth.getUserProfile()
  .then(user => {
    console.log('User profile:', user)
    // Update UI with user data
  })
  .catch(error => {
    console.error('Failed to get profile:', error)
  })
```

## Karma System Integration

### Add Karma Points

```javascript
auth.addKarma(10, 'post_created', 'User created a helpful post')
  .then(result => {
    console.log('Karma added:', result)
    console.log('New total:', result.newKarmaTotal)
    
    // Update UI
    document.getElementById('karma-count').textContent = result.newKarmaTotal
  })
  .catch(error => {
    console.error('Failed to add karma:', error)
  })
```

### Karma Actions Reference

```javascript
// Common karma actions
const KARMA_ACTIONS = {
  POST_CREATED: 'post_created',           // +10 points
  POST_LIKED: 'post_liked',               // +2 points
  COMMENT_POSTED: 'comment_posted',       // +5 points
  PROFILE_COMPLETED: 'profile_completed', // +20 points
  DAILY_LOGIN: 'daily_login',             // +1 point
  ACHIEVEMENT_UNLOCKED: 'achievement'     // Variable points
}

// Usage
auth.addKarma(5, KARMA_ACTIONS.COMMENT_POSTED, 'Posted insightful comment')
```

## Framework Integration

### React Integration

```jsx
import { useEffect, useState } from 'react'

function AuthProvider({ children }) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    const auth = new AhamAuth({
      clientId: process.env.REACT_APP_AHAM_CLIENT_ID,
      subdomain: process.env.REACT_APP_SUBDOMAIN,
      onLogin: (user) => {
        setUser(user)
        setLoading(false)
      },
      onLogout: () => {
        setUser(null)
        setLoading(false)
      },
      onError: (error) => {
        console.error('Auth error:', error)
        setLoading(false)
      }
    })
    
    // Store auth instance globally or in context
    window.ahamAuth = auth
  }, [])
  
  if (loading) return <div>Loading...</div>
  
  return (
    <AuthContext.Provider value={{ user, auth: window.ahamAuth }}>
      {children}
    </AuthContext.Provider>
  )
}

// Usage in components
function LoginButton() {
  const { auth } = useContext(AuthContext)
  
  return (
    <button onClick={() => auth.login({ popup: true })}>
      Login with Aham
    </button>
  )
}
```

### Vue.js Integration

```javascript
// Vue plugin
const AhamAuthPlugin = {
  install(app, options) {
    const auth = new AhamAuth(options)
    
    app.config.globalProperties.$auth = auth
    app.provide('auth', auth)
  }
}

// Main.js
app.use(AhamAuthPlugin, {
  clientId: process.env.VUE_APP_AHAM_CLIENT_ID,
  subdomain: process.env.VUE_APP_SUBDOMAIN
})

// Component usage
export default {
  inject: ['auth'],
  methods: {
    login() {
      this.auth.login({ popup: true })
    }
  }
}
```

## Error Handling

### Common Error Scenarios

```javascript
const auth = new AhamAuth({
  clientId: 'your_client_id',
  subdomain: 'your_subdomain',
  
  onError: (error) => {
    switch (error.message) {
      case 'Invalid token':
        // Token expired or invalid
        auth.logout()
        showLoginPrompt()
        break
        
      case 'Login popup was closed':
        // User closed popup without completing login
        showMessage('Login was cancelled', 'info')
        break
        
      case 'Network error':
        // Connection issues
        showMessage('Connection error. Please check your internet.', 'error')
        break
        
      default:
        console.error('Unhandled auth error:', error)
        showMessage('Authentication error occurred', 'error')
    }
  }
})
```

## Best Practices

### Security

1. **Never expose sensitive config in client code**
```javascript
// ❌ Bad
const auth = new AhamAuth({
  clientSecret: 'secret_key' // Never do this!
})

// ✅ Good
const auth = new AhamAuth({
  clientId: 'public_client_id' // Only public identifiers
})
```

2. **Use HTTPS in production**
```javascript
const auth = new AhamAuth({
  authDomain: location.protocol === 'https:' 
    ? 'https://aham.brah.ma' 
    : 'http://localhost:3000'
})
```

### Performance

1. **Initialize SDK early**
```html
<!-- Load SDK in <head> for faster initialization -->
<head>
  <script src="https://aham.brah.ma/aham-auth-sdk.js" defer></script>
</head>
```

2. **Cache user data appropriately**
```javascript
// Cache non-sensitive user data
const user = auth.getUser()
if (user) {
  sessionStorage.setItem('user_display_name', user.name)
}
```

## Troubleshooting

### Common Issues

1. **SDK not loading**
   - Check network connectivity
   - Verify CDN URL is correct
   - Check browser console for errors

2. **Login popup blocked**
   - Ensure popup is triggered by user action
   - Check browser popup settings
   - Use redirect flow as fallback

3. **Token validation failing**
   - Check client ID configuration
   - Verify domain settings
   - Check server logs for errors

4. **CORS errors**
   - Ensure domain is allowlisted
   - Check protocol (HTTP vs HTTPS)
   - Verify subdomain configuration

### Debug Mode

```javascript
const auth = new AhamAuth({
  debug: true, // Enable detailed logging
  clientId: 'your_client_id',
  subdomain: 'your_subdomain'
})

// View debug information
console.log('Auth config:', auth)
console.log('Current session:', auth.getSession())
```

## Support and Resources

- **Documentation**: [https://docs.brah.ma](https://docs.brah.ma)
- **API Reference**: See `API_REFERENCE.md`
- **GitHub Issues**: Repository issues
- **Email Support**: dev@brah.ma 