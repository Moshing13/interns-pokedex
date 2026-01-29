# 07 - Building Routes

Routes connect URLs to controllers. They're the "address book" of your application.

## What is a Route?

When someone visits `http://localhost:3000/pokemon/pikachu`:
1. Express checks all routes
2. Finds the matching pattern (`/pokemon/:nameOrId`)
3. Calls the associated controller function

## Express Router Basics

```javascript
import { Router } from 'express';

const router = Router();

// Define routes
router.get('/path', controllerFunction);

export default router;
```

## Route Patterns

### Static Routes

```javascript
router.get('/about', aboutController);
// Matches: /about
// Does NOT match: /about/team
```

### Dynamic Routes (Parameters)

```javascript
router.get('/pokemon/:nameOrId', pokemonController);
// Matches: /pokemon/pikachu, /pokemon/25, /pokemon/mr-mime
// Access via: req.params.nameOrId
```

### Multiple Parameters

```javascript
router.get('/trainer/:trainerId/pokemon/:pokemonId', handler);
// Matches: /trainer/ash/pokemon/25
// Access: req.params.trainerId, req.params.pokemonId
```

## Creating the Routes

### Main Router

Create `src/routes/index.js`:

```javascript
import { Router } from 'express';
import pokemonRoutes from './pokemonRoutes.js';

const router = Router();

// Mount all Pokemon routes at root
router.use('/', pokemonRoutes);

export default router;
```

This file is the main entry point for all routes. It can include routes from multiple files.

### Pokemon Routes

Create `src/routes/pokemonRoutes.js`:

```javascript
import { Router } from 'express';
import * as pokemonController from '../controllers/pokemonController.js';

const router = Router();

// ============================================
// VIEW ROUTES (Return HTML)
// ============================================

// Home page - list all Pokemon
router.get('/', pokemonController.getHomePage);

// Search Pokemon
router.get('/search', pokemonController.searchPokemon);

// Filter by type
router.get('/type/:type', pokemonController.getPokemonByType);

// Pokemon detail page
router.get('/pokemon/:nameOrId', pokemonController.getPokemonDetails);

// ============================================
// API ROUTES (Return JSON)
// ============================================

// Get all Pokemon (paginated)
router.get('/api/pokemon', pokemonController.apiGetAllPokemon);

// Search Pokemon
router.get('/api/pokemon/search', pokemonController.apiSearchPokemon);

// Get single Pokemon
router.get('/api/pokemon/:nameOrId', pokemonController.apiGetPokemonDetails);

// Get all types
router.get('/api/types', pokemonController.apiGetTypes);

// Get Pokemon by type
router.get('/api/types/:type', pokemonController.apiGetPokemonByType);

export default router;
```

## Route Order Matters!

Routes are matched in order. More specific routes should come before generic ones:

```javascript
// CORRECT ORDER ✓
router.get('/api/pokemon/search', searchController);   // Specific
router.get('/api/pokemon/:nameOrId', detailController); // Generic

// WRONG ORDER ✗
router.get('/api/pokemon/:nameOrId', detailController); // This would match first!
router.get('/api/pokemon/search', searchController);     // Never reached!
```

Why? Because `/api/pokemon/search` would match `:nameOrId` as "search".

## Complete Route Reference

### View Routes (HTML)

| Method | Route | Controller | Description |
|--------|-------|------------|-------------|
| GET | `/` | `getHomePage` | Home page with Pokemon list |
| GET | `/search` | `searchPokemon` | Search results page |
| GET | `/type/:type` | `getPokemonByType` | Pokemon filtered by type |
| GET | `/pokemon/:nameOrId` | `getPokemonDetails` | Pokemon detail page |

### API Routes (JSON)

| Method | Route | Controller | Description |
|--------|-------|------------|-------------|
| GET | `/api/pokemon` | `apiGetAllPokemon` | List Pokemon (paginated) |
| GET | `/api/pokemon/search` | `apiSearchPokemon` | Search Pokemon |
| GET | `/api/pokemon/:nameOrId` | `apiGetPokemonDetails` | Get single Pokemon |
| GET | `/api/types` | `apiGetTypes` | List all types |
| GET | `/api/types/:type` | `apiGetPokemonByType` | Pokemon by type |

## Query Strings

Query strings (`?key=value`) are NOT part of route matching:

```javascript
router.get('/search', handler);
// Matches: /search, /search?q=pikachu, /search?q=pikachu&limit=10
// Access query: req.query.q, req.query.limit
```

## HTTP Methods

While we only use GET in this app, here are all methods:

```javascript
router.get('/items', getItems);       // Read
router.post('/items', createItem);    // Create
router.put('/items/:id', updateItem); // Update (replace)
router.patch('/items/:id', patchItem); // Update (partial)
router.delete('/items/:id', deleteItem); // Delete
```

## Route Organization Tips

### 1. Group Related Routes

Keep routes for the same resource together:

```javascript
// All Pokemon routes
router.get('/pokemon', listPokemon);
router.get('/pokemon/:id', getPokemon);
router.post('/pokemon', createPokemon);
```

### 2. Use Separate Files for Large Apps

```
src/routes/
├── index.js          # Main router (imports others)
├── pokemonRoutes.js  # /pokemon routes
├── trainerRoutes.js  # /trainer routes
└── authRoutes.js     # /auth routes
```

### 3. Prefix API Routes

```javascript
// Clear distinction between views and API
router.get('/pokemon/:id', renderPokemonPage);     // HTML
router.get('/api/pokemon/:id', getPokemonJSON);    // JSON
```

## How Routes Connect to the App

In `src/app.js`, routes are connected using `app.use()`:

```javascript
import routes from './routes/index.js';

const app = express();

// ... middleware setup ...

// Connect all routes
app.use('/', routes);
```

You can also add a prefix:

```javascript
app.use('/v1', routes);  // All routes now start with /v1
// /pokemon becomes /v1/pokemon
```

## Testing Routes

Use curl or your browser:

```bash
# Home page
curl http://localhost:3000/

# Pokemon detail
curl http://localhost:3000/pokemon/pikachu

# API endpoint
curl http://localhost:3000/api/pokemon?limit=5

# Search with query
curl "http://localhost:3000/api/pokemon/search?q=char"
```

## Common Mistakes

### 1. Forgetting to Export

```javascript
// WRONG - router not exported
const router = Router();
router.get('/', handler);

// CORRECT
export default router;
```

### 2. Missing Import

```javascript
// WRONG - Router not imported
const router = Router();

// CORRECT
import { Router } from 'express';
const router = Router();
```

### 3. Wrong Order

```javascript
// WRONG - dynamic before static
router.get('/:id', handler);
router.get('/new', handler);  // Never reached!

// CORRECT
router.get('/new', handler);   // Specific first
router.get('/:id', handler);   // Generic last
```

## Summary

- Routes map URLs to controller functions
- Use `:param` for dynamic URL segments
- Order matters - specific routes before generic
- Query strings are accessed via `req.query`
- Export routers and import them in the main app

## Commit Your Progress

Commit your routes:

```bash
git add .
git commit -m "feat: add application routes

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## What's Next?

Now let's create the EJS templates to display our data!

Next: [08 - Building Views](./08-building-the-views.md)
