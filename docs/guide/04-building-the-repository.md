# 04 - Building the Repository Layer

The repository layer is responsible for fetching data from external sources. In our case, it talks to the PokeAPI.

## What is a Repository?

A repository is like a librarian:
- You ask for a book (data)
- The librarian knows where to find it (API endpoint)
- They retrieve and give it to you (return data)

Other parts of your app don't need to know HOW data is fetched - they just ask the repository.

## Understanding PokeAPI

Before we write code, let's understand the API we're using.

### Key Endpoints

| Endpoint | Description | Example |
|----------|-------------|---------|
| `/pokemon` | List all Pokemon | `/pokemon?limit=20&offset=0` |
| `/pokemon/{name}` | Get Pokemon details | `/pokemon/pikachu` |
| `/pokemon-species/{name}` | Get species info | `/pokemon-species/pikachu` |
| `/type` | List all types | `/type` |
| `/type/{name}` | Pokemon by type | `/type/electric` |

### Example API Response

When we call `/pokemon/pikachu`:

```json
{
  "id": 25,
  "name": "pikachu",
  "height": 4,
  "weight": 60,
  "types": [
    { "type": { "name": "electric" } }
  ],
  "stats": [
    { "base_stat": 35, "stat": { "name": "hp" } },
    { "base_stat": 55, "stat": { "name": "attack" } }
  ],
  "sprites": {
    "front_default": "https://..."
  }
}
```

## Creating the Repository

Create `src/repositories/pokemonRepository.js`:

```javascript
import axios from 'axios';
import { config } from '../config/index.js';

// Get the base URL from config
const { baseUrl: BASE_URL } = config.pokeapi;

/**
 * Fetch a paginated list of all Pokemon
 * @param {number} limit - Number of Pokemon to fetch
 * @param {number} offset - Starting position
 * @returns {Promise<Object>} - List of Pokemon with count
 */
export const getAllPokemon = async (limit = 20, offset = 0) => {
  try {
    const response = await axios.get(`${BASE_URL}/pokemon`, {
      params: { limit, offset }
    });
    return response.data;
  } catch (error) {
    throw new Error(`Failed to fetch Pokemon list: ${error.message}`);
  }
};

/**
 * Fetch a single Pokemon by name or ID
 * @param {string|number} nameOrId - Pokemon name or ID
 * @returns {Promise<Object|null>} - Pokemon data or null if not found
 */
export const getPokemonByNameOrId = async (nameOrId) => {
  try {
    const response = await axios.get(
      `${BASE_URL}/pokemon/${nameOrId.toString().toLowerCase()}`
    );
    return response.data;
  } catch (error) {
    // Return null for 404 (not found) instead of throwing
    if (error.response && error.response.status === 404) {
      return null;
    }
    throw new Error(`Failed to fetch Pokemon: ${error.message}`);
  }
};

/**
 * Fetch Pokemon species data (for descriptions)
 * @param {string|number} nameOrId - Pokemon name or ID
 * @returns {Promise<Object|null>} - Species data or null if not found
 */
export const getPokemonSpecies = async (nameOrId) => {
  try {
    const response = await axios.get(
      `${BASE_URL}/pokemon-species/${nameOrId.toString().toLowerCase()}`
    );
    return response.data;
  } catch (error) {
    if (error.response && error.response.status === 404) {
      return null;
    }
    throw new Error(`Failed to fetch Pokemon species: ${error.message}`);
  }
};

/**
 * Search Pokemon by name
 * @param {string} query - Search query
 * @param {number} limit - Maximum results to search through
 * @returns {Promise<Object>} - Filtered Pokemon list
 */
export const searchPokemon = async (query, limit = config.pagination.maxSearchLimit) => {
  try {
    // Fetch all Pokemon up to the limit
    const response = await axios.get(`${BASE_URL}/pokemon`, {
      params: { limit, offset: 0 }
    });

    // Filter by name locally
    const allPokemon = response.data.results;
    const filtered = allPokemon.filter((pokemon) =>
      pokemon.name.toLowerCase().includes(query.toLowerCase())
    );

    return {
      count: filtered.length,
      results: filtered
    };
  } catch (error) {
    throw new Error(`Failed to search Pokemon: ${error.message}`);
  }
};

/**
 * Fetch all Pokemon types
 * @returns {Promise<Array>} - List of types
 */
export const getPokemonTypes = async () => {
  try {
    const response = await axios.get(`${BASE_URL}/type`);
    return response.data.results;
  } catch (error) {
    throw new Error(`Failed to fetch Pokemon types: ${error.message}`);
  }
};

/**
 * Fetch all Pokemon of a specific type
 * @param {string} typeName - Type name (e.g., "electric")
 * @returns {Promise<Array|null>} - List of Pokemon or null if type not found
 */
export const getPokemonByType = async (typeName) => {
  try {
    const response = await axios.get(`${BASE_URL}/type/${typeName.toLowerCase()}`);
    // Extract just the Pokemon info from the nested structure
    return response.data.pokemon.map((p) => p.pokemon);
  } catch (error) {
    if (error.response && error.response.status === 404) {
      return null;
    }
    throw new Error(`Failed to fetch Pokemon by type: ${error.message}`);
  }
};
```

## Code Breakdown

### Importing Dependencies

```javascript
import axios from 'axios';
import { config } from '../config/index.js';

const { baseUrl: BASE_URL } = config.pokeapi;
```

- **axios**: Library for making HTTP requests
- **config**: Our configuration object
- We extract `baseUrl` and rename it to `BASE_URL` (using destructuring)

### The getAllPokemon Function

```javascript
export const getAllPokemon = async (limit = 20, offset = 0) => {
  try {
    const response = await axios.get(`${BASE_URL}/pokemon`, {
      params: { limit, offset }
    });
    return response.data;
  } catch (error) {
    throw new Error(`Failed to fetch Pokemon list: ${error.message}`);
  }
};
```

Key points:
- `async` function - returns a Promise
- `await axios.get()` - wait for the HTTP request to complete
- `params` object - axios adds these as query string (`?limit=20&offset=0`)
- `try/catch` - handle errors gracefully
- `throw new Error()` - pass error up with a clear message

### Handling 404 Errors

```javascript
if (error.response && error.response.status === 404) {
  return null;
}
```

For "not found" errors, we return `null` instead of throwing. This lets the caller decide how to handle missing data.

### The Search Function

```javascript
export const searchPokemon = async (query, limit = config.pagination.maxSearchLimit) => {
  // Fetch all Pokemon
  const response = await axios.get(`${BASE_URL}/pokemon`, {
    params: { limit, offset: 0 }
  });

  // Filter locally
  const filtered = allPokemon.filter((pokemon) =>
    pokemon.name.toLowerCase().includes(query.toLowerCase())
  );
```

PokeAPI doesn't have a search endpoint, so we:
1. Fetch all Pokemon (up to a limit)
2. Filter locally using JavaScript's `filter()` method
3. Use `toLowerCase()` for case-insensitive matching

## Using Axios

### Making GET Requests

```javascript
// Simple GET
const response = await axios.get('https://api.example.com/data');

// GET with query parameters
const response = await axios.get('https://api.example.com/data', {
  params: { limit: 10, page: 1 }
});
// Results in: https://api.example.com/data?limit=10&page=1
```

### Understanding the Response

```javascript
const response = await axios.get(url);

console.log(response.data);    // The actual data
console.log(response.status);  // HTTP status code (200, 404, etc.)
console.log(response.headers); // Response headers
```

## Error Handling Patterns

### Pattern 1: Throw All Errors

```javascript
export const getData = async () => {
  const response = await axios.get(url);
  return response.data;
};
// Errors bubble up - caller must handle
```

### Pattern 2: Handle Specific Errors

```javascript
export const getData = async () => {
  try {
    const response = await axios.get(url);
    return response.data;
  } catch (error) {
    if (error.response?.status === 404) {
      return null; // Not found is OK
    }
    throw error; // Other errors bubble up
  }
};
```

### Pattern 3: Return Error Object

```javascript
export const getData = async () => {
  try {
    const response = await axios.get(url);
    return { success: true, data: response.data };
  } catch (error) {
    return { success: false, error: error.message };
  }
};
```

We use Pattern 2 in our app - specific errors are handled, others are thrown.

## Testing the Repository

You can test your repository functions directly:

```javascript
// test-repository.js (temporary test file)
import * as pokemonRepository from './src/repositories/pokemonRepository.js';

async function test() {
  console.log('Testing getAllPokemon...');
  const list = await pokemonRepository.getAllPokemon(5, 0);
  console.log('Got', list.results.length, 'Pokemon');

  console.log('\nTesting getPokemonByNameOrId...');
  const pikachu = await pokemonRepository.getPokemonByNameOrId('pikachu');
  console.log('Got', pikachu.name);

  console.log('\nTesting searchPokemon...');
  const search = await pokemonRepository.searchPokemon('char');
  console.log('Found', search.count, 'Pokemon matching "char"');
}

test().catch(console.error);
```

Run it:

```bash
node test-repository.js
```

## Summary

| Function | Purpose | Returns |
|----------|---------|---------|
| `getAllPokemon(limit, offset)` | Get paginated list | `{ count, results }` |
| `getPokemonByNameOrId(nameOrId)` | Get single Pokemon | Pokemon object or `null` |
| `getPokemonSpecies(nameOrId)` | Get species data | Species object or `null` |
| `searchPokemon(query)` | Search by name | `{ count, results }` |
| `getPokemonTypes()` | List all types | Array of types |
| `getPokemonByType(typeName)` | Pokemon by type | Array or `null` |

## Commit Your Progress

Commit your repository layer:

```bash
git add .
git commit -m "feat: add pokemon repository for API calls

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## What's Next?

Now that we can fetch data, let's create the service layer to transform it.

Next: [05 - Building the Service Layer](./05-building-the-service.md)
