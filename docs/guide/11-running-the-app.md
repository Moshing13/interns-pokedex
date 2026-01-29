# 11 - Running the App

Congratulations! You've built a complete Pokedex application. Let's put everything together and run it!

## The Main Entry Point

Create `src/app.js` - this is where everything comes together:

```javascript
import express from 'express';
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';
import { config } from './config/index.js';
import routes from './routes/index.js';

// ES Modules don't have __dirname by default
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// Create Express app
const app = express();
const { port: PORT, nodeEnv } = config;

// ============================================
// MIDDLEWARE
// ============================================

// Parse JSON request bodies
app.use(express.json());

// Parse URL-encoded form data
app.use(express.urlencoded({ extended: true }));

// Serve static files (CSS, images) from public folder
app.use(express.static(join(__dirname, '../public')));

// ============================================
// VIEW ENGINE
// ============================================

// Use EJS as the template engine
app.set('view engine', 'ejs');

// Set the views directory
app.set('views', join(__dirname, 'views'));

// ============================================
// ROUTES
// ============================================

// Mount all routes
app.use('/', routes);

// ============================================
// ERROR HANDLERS
// ============================================

// 404 - Not Found
app.use((req, res) => {
  res.status(404).render('error', {
    message: 'Page not found',
    error: 'The page you are looking for does not exist.'
  });
});

// 500 - Server Error
app.use((err, _req, res, _next) => {
  res.status(500).render('error', {
    message: 'Something went wrong',
    error: err.message
  });
});

// ============================================
// START SERVER
// ============================================

// Only start the server if not in test mode
if (nodeEnv !== 'test') {
  app.listen(PORT, () => {
    console.log(`Pokedex server running at http://localhost:${PORT}`);
  });
}

// Export for testing
export default app;
```

## Understanding app.js

### 1. Middleware Setup

```javascript
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static(join(__dirname, '../public')));
```

| Middleware | Purpose |
|------------|---------|
| `express.json()` | Parse JSON in request body |
| `express.urlencoded()` | Parse form data |
| `express.static()` | Serve static files (CSS, images) |

### 2. View Engine

```javascript
app.set('view engine', 'ejs');
app.set('views', join(__dirname, 'views'));
```

Tells Express:
- Use EJS to render templates
- Templates are in the `views` folder

### 3. Error Handlers

Error handlers have 4 parameters: `(err, req, res, next)`

```javascript
app.use((err, _req, res, _next) => {
  res.status(500).render('error', { ... });
});
```

The underscore prefix (`_req`) indicates unused parameters.

## Running the Application

### Development Mode

```bash
npm run dev
```

This uses `nodemon` to automatically restart when you make changes.

Output:
```
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,cjs,json
[nodemon] starting `node src/app.js`
Pokedex server running at http://localhost:3000
```

### Production Mode

```bash
npm start
```

Runs without auto-reload.

## Using the Application

Open your browser to `http://localhost:3000`

### Features to Try

1. **Home Page** - View the Pokemon grid
2. **Pagination** - Click "Next" to see more Pokemon
3. **Search** - Type a Pokemon name and search
4. **Filter by Type** - Click a type button to filter
5. **Detail Page** - Click a Pokemon card to see details

### API Endpoints

Test the JSON API with curl or your browser:

```bash
# Get Pokemon list
curl http://localhost:3000/api/pokemon

# Get specific Pokemon
curl http://localhost:3000/api/pokemon/pikachu

# Search
curl "http://localhost:3000/api/pokemon/search?q=char"

# Get types
curl http://localhost:3000/api/types

# Get Pokemon by type
curl http://localhost:3000/api/types/fire
```

## Running Tests

```bash
# Run all tests with coverage
npm test

# Expected output:
# PASS  tests/pokemonService.test.js
# PASS  tests/api.test.js
# Test Suites: 2 passed, 2 total
# Coverage: ~85%
```

## Code Quality Checks

```bash
# Check for linting errors
npm run lint

# Fix linting errors automatically
npm run lint:fix

# Check code formatting
npm run format:check

# Format code
npm run format
```

## Troubleshooting

### "Cannot find module" Error

Make sure you've installed dependencies:
```bash
npm install
```

### "Port 3000 is already in use"

Change the port in `.env`:
```
PORT=3001
```

Or kill the process using port 3000:
```bash
# On Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# On Mac/Linux
lsof -i :3000
kill -9 <PID>
```

### API Errors

If PokeAPI is down or slow:
- The app will show error pages
- Try again in a few minutes
- Check https://pokeapi.co for status

### Template Errors

Check that:
- All EJS files are in `src/views/`
- Partials are in `src/views/partials/`
- Variable names match between controller and template

## Final Project Structure

```
pokedex-app/
├── src/
│   ├── app.js                  # Main entry point
│   ├── config/
│   │   └── index.js            # Configuration
│   ├── routes/
│   │   ├── index.js            # Main router
│   │   └── pokemonRoutes.js    # Pokemon routes
│   ├── controllers/
│   │   └── pokemonController.js
│   ├── services/
│   │   └── pokemonService.js
│   ├── repositories/
│   │   └── pokemonRepository.js
│   └── views/
│       ├── index.ejs
│       ├── pokemon.ejs
│       ├── error.ejs
│       └── partials/
│           ├── header.ejs
│           └── footer.ejs
├── public/
│   └── css/
│       └── style.css
├── tests/
│   ├── api.test.js
│   └── pokemonService.test.js
├── .env
├── .env.example
├── .gitignore
├── package.json
├── jest.config.js
└── README.md
```

## What You've Learned

- Setting up a Node.js project with Express
- Layered architecture (Routes → Controllers → Services → Repositories)
- Server-side rendering with EJS templates
- Making HTTP requests with Axios
- Environment configuration with dotenv
- CSS Grid and responsive design
- Testing with Jest and Supertest
- Code quality with ESLint and Prettier

## Next Steps

Ideas to extend your Pokedex:

1. **Add more features**
   - Evolution chain display
   - Pokemon comparison
   - Favorites list (with localStorage)

2. **Improve performance**
   - Add caching (Redis or in-memory)
   - Lazy load images
   - Pagination in URL (for bookmarking)

3. **Add more tests**
   - Test edge cases
   - Add integration tests
   - Set up CI/CD with GitHub Actions

4. **Deploy your app**
   - Heroku
   - Vercel
   - Railway

## Commit Your Progress

Commit the main application entry point:

```bash
git add .
git commit -m "feat: add express application entry point

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## Push Your Branch

Now that you've completed all sections, push your branch to GitHub:

```bash
git push -u origin your-lastname/pokedex-pull-request
```

Replace `your-lastname/pokedex-pull-request` with your actual branch name (e.g., `dela-cruz/pokedex-pull-request`).

## Create a Pull Request

Submit your work by creating a pull request:

1. Go to the original repository on GitHub
2. Click **Pull Requests** → **New Pull Request**
3. Click **compare across forks**
4. Select your fork and branch
5. Fill in the PR details:

**Title:** `feat: complete pokedex application`

**Description:**
```
## Summary
- Implemented Pokemon list with pagination
- Added search functionality
- Added type filtering
- Created detail pages for each Pokemon
- Added responsive CSS styling

## Checklist
- [ ] All tests pass (`npm test`)
- [ ] Code is formatted (`npm run format`)
- [ ] No linting errors (`npm run lint`)

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com
```

6. Click **Create Pull Request**

## Conclusion

You've built a complete full-stack web application! The skills you've learned here apply to many types of projects. Keep practicing and building!

Happy coding!
