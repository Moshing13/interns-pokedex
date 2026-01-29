# 03 - Configuration

In this section, we'll set up our application's configuration. Good configuration management makes your app flexible and secure.

## Why Configuration Matters

Configuration separates settings from code:
- **Different environments** (development vs production) can use different settings
- **Sensitive data** (API keys, passwords) stay out of your code
- **Easy changes** without modifying code

## Environment Variables

Environment variables are settings stored outside your code. They're loaded at runtime.

### The `.env` File

Create a `.env` file in your project root:

```env
# Server Configuration
PORT=3000
NODE_ENV=development

# PokeAPI Configuration
POKEAPI_BASE_URL=https://pokeapi.co/api/v2

# Pagination Settings
DEFAULT_PAGE_LIMIT=20
MAX_SEARCH_LIMIT=1000
```

### What Each Variable Does

| Variable | Default | Purpose |
|----------|---------|---------|
| `PORT` | 3000 | Which port the server runs on |
| `NODE_ENV` | development | Environment mode (development/production) |
| `POKEAPI_BASE_URL` | https://pokeapi.co/api/v2 | Base URL for Pokemon API |
| `DEFAULT_PAGE_LIMIT` | 20 | Pokemon per page |
| `MAX_SEARCH_LIMIT` | 1000 | Maximum Pokemon to search through |

## The Config File

Now let's create a config file that loads and organizes these variables.

Create `src/config/index.js`:

```javascript
import dotenv from 'dotenv';

// Load environment variables from .env file
dotenv.config();

export const config = {
  // Server settings
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',

  // PokeAPI settings
  pokeapi: {
    baseUrl: process.env.POKEAPI_BASE_URL || 'https://pokeapi.co/api/v2'
  },

  // Pagination settings
  pagination: {
    defaultLimit: parseInt(process.env.DEFAULT_PAGE_LIMIT, 10) || 20,
    maxSearchLimit: parseInt(process.env.MAX_SEARCH_LIMIT, 10) || 1000
  }
};
```

### Code Breakdown

Let's understand each part:

#### 1. Import dotenv

```javascript
import dotenv from 'dotenv';
```

The `dotenv` package reads your `.env` file and loads variables into `process.env`.

#### 2. Load the variables

```javascript
dotenv.config();
```

This line reads the `.env` file. After this, you can access variables like `process.env.PORT`.

#### 3. Export the config object

```javascript
export const config = {
  port: process.env.PORT || 3000,
  // ...
};
```

We create a structured object with all our settings. The `||` operator provides fallback values if variables aren't set.

#### 4. Parse numbers

```javascript
defaultLimit: parseInt(process.env.DEFAULT_PAGE_LIMIT, 10) || 20,
```

Environment variables are always strings. We use `parseInt()` to convert them to numbers.
- `10` is the radix (base 10)
- `|| 20` provides a fallback if parsing fails

## Using the Config

Here's how other files use the config:

```javascript
// In any file that needs configuration
import { config } from './config/index.js';

// Use the settings
console.log(`Server running on port ${config.port}`);
console.log(`API URL: ${config.pokeapi.baseUrl}`);
console.log(`Items per page: ${config.pagination.defaultLimit}`);
```

## Package.json Configuration

Make sure your `package.json` has the correct module type:

```json
{
  "type": "module"
}
```

This enables ES6 `import`/`export` syntax instead of `require()`.

## Best Practices

### 1. Always Use Fallback Values

```javascript
// Good - has fallback
port: process.env.PORT || 3000

// Bad - might be undefined
port: process.env.PORT
```

### 2. Never Commit `.env`

Add `.env` to your `.gitignore`:

```
.env
.env.local
.env.*.local
```

### 3. Provide an Example File

Create `.env.example` with placeholder values:

```env
PORT=3000
NODE_ENV=development
POKEAPI_BASE_URL=https://pokeapi.co/api/v2
DEFAULT_PAGE_LIMIT=20
MAX_SEARCH_LIMIT=1000
```

This shows other developers what variables they need without exposing real values.

### 4. Validate Required Variables

For critical settings, you can add validation:

```javascript
// Optional: Validate required variables
if (!process.env.POKEAPI_BASE_URL) {
  console.warn('Warning: POKEAPI_BASE_URL not set, using default');
}
```

## Common Configuration Patterns

### Group Related Settings

```javascript
export const config = {
  // Group server settings
  server: {
    port: process.env.PORT || 3000,
    env: process.env.NODE_ENV || 'development'
  },

  // Group API settings
  api: {
    baseUrl: process.env.API_URL,
    timeout: parseInt(process.env.API_TIMEOUT, 10) || 5000
  }
};
```

### Environment-Specific Settings

```javascript
const isDev = process.env.NODE_ENV === 'development';

export const config = {
  debug: isDev,
  logLevel: isDev ? 'debug' : 'error'
};
```

## Testing Your Config

Create a simple test file to verify your config loads correctly:

```javascript
// test-config.js (temporary file for testing)
import { config } from './src/config/index.js';

console.log('Configuration loaded:');
console.log(JSON.stringify(config, null, 2));
```

Run it with:

```bash
node test-config.js
```

Expected output:

```json
{
  "port": 3000,
  "nodeEnv": "development",
  "pokeapi": {
    "baseUrl": "https://pokeapi.co/api/v2"
  },
  "pagination": {
    "defaultLimit": 20,
    "maxSearchLimit": 1000
  }
}
```

## Summary

| File | Purpose |
|------|---------|
| `.env` | Store environment variables (not committed) |
| `.env.example` | Template showing required variables |
| `src/config/index.js` | Load and organize configuration |

## Commit Your Progress

Commit your configuration setup:

```bash
git add .
git commit -m "feat: add application configuration

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## What's Next?

With configuration set up, let's build the repository layer - the part that talks to PokeAPI.

Next: [04 - Building the Repository Layer](./04-building-the-repository.md)
