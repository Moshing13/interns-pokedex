# 06 - Building the Controller Layer

Controllers handle HTTP requests and responses. They're the bridge between your routes (URLs) and your services (business logic).

## What is a Controller?

Think of controllers as customer service representatives:
- They receive requests from customers (HTTP requests)
- They get the information needed (call services)
- They respond in the right format (HTML or JSON)

## Two Types of Controllers

Our app has two types of controllers:

| Type | Output | Use Case |
|------|--------|----------|
| **View Controllers** | HTML pages (via EJS) | Web browsers |
| **API Controllers** | JSON data | Mobile apps, other services |

## Understanding Request and Response

Every Express controller receives two objects:

```javascript
function controller(req, res) {
  // req = incoming request information
  // res = methods to send responses
}
```

### The Request Object (req)

```javascript
// URL: /pokemon/pikachu?format=detailed

req.params    // { nameOrId: "pikachu" }  - URL parameters
req.query     // { format: "detailed" }    - Query string
req.body      // { ... }                   - POST body data
req.method    // "GET"                     - HTTP method
```

### The Response Object (res)

```javascript
// For HTML responses
res.render('template', { data });  // Render EJS template

// For JSON responses
res.json({ success: true, data }); // Send JSON

// Set status code
res.status(404).render('error');   // Set status + render
res.status(500).json({ error });   // Set status + send JSON
```

## Creating the Controller

Create `src/controllers/pokemonController.js`:

```javascript
import * as pokemonService from '../services/pokemonService.js';

// ============================================
// VIEW CONTROLLERS (Return HTML via EJS)
// ============================================

/**
 * Home page - List all Pokemon with pagination
 */
export const getHomePage = async (req, res) => {
  try {
    // Get pagination parameters from query string
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 20;

    // Fetch data from services
    const data = await pokemonService.getAllPokemon(page, limit);
    const types = await pokemonService.getPokemonTypes();

    // Render the index template
    res.render('index', {
      ...data,          // Spread pokemon data
      types,            // Add types for filter dropdown
      searchQuery: '',  // Empty search box
      selectedType: ''  // No type selected
    });
  } catch (error) {
    // Render error page on failure
    res.status(500).render('error', {
      message: 'Failed to load Pokemon',
      error: error.message
    });
  }
};

/**
 * Pokemon detail page
 */
export const getPokemonDetails = async (req, res) => {
  try {
    // Get Pokemon name/id from URL parameter
    const { nameOrId } = req.params;

    // Fetch Pokemon details
    const pokemon = await pokemonService.getPokemonDetails(nameOrId);

    // Handle not found
    if (!pokemon) {
      return res.status(404).render('error', {
        message: 'Pokemon not found',
        error: `No Pokemon found with name or ID: ${nameOrId}`
      });
    }

    // Render detail page
    res.render('pokemon', { pokemon });
  } catch (error) {
    res.status(500).render('error', {
      message: 'Failed to load Pokemon details',
      error: error.message
    });
  }
};

/**
 * Search results page
 */
export const searchPokemon = async (req, res) => {
  try {
    // Get search query from URL (?q=pikachu)
    const { q } = req.query;

    // Fetch types (for filter dropdown) and search results
    const types = await pokemonService.getPokemonTypes();
    const data = await pokemonService.searchPokemon(q);

    // Render using same index template
    res.render('index', {
      ...data,
      types,
      currentPage: 1,
      totalPages: 1,
      hasNextPage: false,
      hasPrevPage: false,
      searchQuery: q || '',  // Show search term in box
      selectedType: ''
    });
  } catch (error) {
    res.status(500).render('error', {
      message: 'Search failed',
      error: error.message
    });
  }
};

/**
 * Filter by type page
 */
export const getPokemonByType = async (req, res) => {
  try {
    const { type } = req.params;
    const page = parseInt(req.query.page) || 1;

    const types = await pokemonService.getPokemonTypes();
    const data = await pokemonService.getPokemonByType(type, page);

    // Handle invalid type
    if (!data) {
      return res.status(404).render('error', {
        message: 'Type not found',
        error: `No Pokemon type found: ${type}`
      });
    }

    res.render('index', {
      ...data,
      types,
      searchQuery: '',
      selectedType: type  // Highlight selected type
    });
  } catch (error) {
    res.status(500).render('error', {
      message: 'Failed to load Pokemon by type',
      error: error.message
    });
  }
};
```

## API Controllers

Add these for JSON responses:

```javascript
// ============================================
// API CONTROLLERS (Return JSON)
// ============================================

/**
 * API: Get all Pokemon
 */
export const apiGetAllPokemon = async (req, res) => {
  try {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 20;
    const data = await pokemonService.getAllPokemon(page, limit);

    res.json({ success: true, data });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

/**
 * API: Get Pokemon by name or ID
 */
export const apiGetPokemonDetails = async (req, res) => {
  try {
    const { nameOrId } = req.params;
    const pokemon = await pokemonService.getPokemonDetails(nameOrId);

    if (!pokemon) {
      return res.status(404).json({
        success: false,
        error: `Pokemon not found: ${nameOrId}`
      });
    }

    res.json({ success: true, data: pokemon });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

/**
 * API: Search Pokemon
 */
export const apiSearchPokemon = async (req, res) => {
  try {
    const { q } = req.query;
    const data = await pokemonService.searchPokemon(q);
    res.json({ success: true, data });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

/**
 * API: Get all types
 */
export const apiGetTypes = async (req, res) => {
  try {
    const types = await pokemonService.getPokemonTypes();
    res.json({ success: true, data: types });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

/**
 * API: Get Pokemon by type
 */
export const apiGetPokemonByType = async (req, res) => {
  try {
    const { type } = req.params;
    const page = parseInt(req.query.page) || 1;
    const data = await pokemonService.getPokemonByType(type, page);

    if (!data) {
      return res.status(404).json({
        success: false,
        error: `Type not found: ${type}`
      });
    }

    res.json({ success: true, data });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};
```

## Understanding Key Patterns

### Pattern 1: Parsing Query Parameters

```javascript
const page = parseInt(req.query.page) || 1;
```

- `req.query.page` is a string (e.g., `"2"`)
- `parseInt()` converts to number
- `|| 1` provides default if parsing fails or undefined

### Pattern 2: Early Return for Errors

```javascript
if (!pokemon) {
  return res.status(404).render('error', { ... });
}
// Continue with normal flow
res.render('pokemon', { pokemon });
```

Using `return` prevents executing the rest of the function.

### Pattern 3: Spread Operator with Data

```javascript
res.render('index', {
  ...data,          // Spreads: { pokemon, currentPage, totalPages, ... }
  types,            // Adds: types
  searchQuery: ''   // Adds: searchQuery
});
```

The spread operator (`...`) unpacks an object's properties.

### Pattern 4: Consistent API Response Format

```javascript
// Success response
res.json({ success: true, data: pokemon });

// Error response
res.status(500).json({ success: false, error: error.message });
```

Always include a `success` field for easy client-side handling.

## HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful request (default) |
| 404 | Not Found | Resource doesn't exist |
| 500 | Server Error | Something went wrong |

## Error Handling

Every controller should have error handling:

```javascript
export const controller = async (req, res) => {
  try {
    // Your logic here
  } catch (error) {
    // Always handle errors!
    res.status(500).render('error', {
      message: 'Something went wrong',
      error: error.message
    });
  }
};
```

## Controller vs Service: What Goes Where?

| Task | Layer | Example |
|------|-------|---------|
| Parse request data | Controller | `parseInt(req.query.page)` |
| Validate input | Controller | `if (!q) return error` |
| Business logic | Service | `formatPokemonData()` |
| Data fetching | Repository | `axios.get()` |
| Send response | Controller | `res.render()` / `res.json()` |

## Testing Controllers

Controllers can be tested using HTTP requests:

```bash
# Test home page
curl http://localhost:3000/

# Test API endpoint
curl http://localhost:3000/api/pokemon/pikachu

# Test search
curl "http://localhost:3000/api/pokemon/search?q=char"
```

## Summary

| Controller | Route | Returns |
|------------|-------|---------|
| `getHomePage` | GET / | HTML (index.ejs) |
| `getPokemonDetails` | GET /pokemon/:nameOrId | HTML (pokemon.ejs) |
| `searchPokemon` | GET /search | HTML (index.ejs) |
| `getPokemonByType` | GET /type/:type | HTML (index.ejs) |
| `apiGetAllPokemon` | GET /api/pokemon | JSON |
| `apiGetPokemonDetails` | GET /api/pokemon/:nameOrId | JSON |
| `apiSearchPokemon` | GET /api/pokemon/search | JSON |
| `apiGetTypes` | GET /api/types | JSON |
| `apiGetPokemonByType` | GET /api/types/:type | JSON |

## Commit Your Progress

Commit your controller layer:

```bash
git add .
git commit -m "feat: add pokemon controllers for request handling

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## What's Next?

Let's connect these controllers to URL routes!

Next: [07 - Building Routes](./07-building-the-routes.md)
