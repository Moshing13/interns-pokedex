# 10 - Testing

Testing ensures your code works correctly and helps prevent bugs. We use Jest as our testing framework.

## Why Test?

- **Catch bugs early** - Find problems before users do
- **Refactor safely** - Change code with confidence
- **Document behavior** - Tests show how code should work
- **Save time** - Automated tests are faster than manual testing

## Testing Setup

### Jest Configuration

Create `jest.config.js`:

```javascript
export default {
  testEnvironment: 'node',
  transform: {},
  moduleFileExtensions: ['js'],
  testMatch: ['**/tests/**/*.test.js'],
  collectCoverageFrom: ['src/**/*.js'],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov'],
  testTimeout: 10000,
  verbose: true
};
```

### Package.json Script

```json
{
  "scripts": {
    "test": "node --experimental-vm-modules node_modules/jest/bin/jest.js --coverage"
  }
}
```

The `--experimental-vm-modules` flag enables ES modules support in Jest.

## Test File Structure

```
tests/
├── api.test.js              # API endpoint tests
├── pokemonService.test.js   # Service layer tests
└── pokemonRepository.test.js # Repository tests
```

## Testing Concepts

### Unit Tests vs Integration Tests

| Type | Tests | Mocks |
|------|-------|-------|
| **Unit** | Single function/module | Dependencies mocked |
| **Integration** | Multiple modules together | Some/no mocks |

We primarily write unit tests with mocked dependencies.

### The AAA Pattern

Every test follows this structure:

```javascript
it('should do something', async () => {
  // ARRANGE - Set up test data and mocks
  mockService.getData.mockResolvedValue({ id: 1 });

  // ACT - Call the code being tested
  const result = await controller.getData();

  // ASSERT - Check the results
  expect(result.id).toBe(1);
});
```

## Testing the Service Layer

Create `tests/pokemonService.test.js`:

```javascript
import { jest } from '@jest/globals';

// Mock the repository BEFORE importing the service
const mockPokemonRepository = {
  getAllPokemon: jest.fn(),
  getPokemonByNameOrId: jest.fn(),
  getPokemonSpecies: jest.fn(),
  searchPokemon: jest.fn(),
  getPokemonTypes: jest.fn(),
  getPokemonByType: jest.fn()
};

jest.unstable_mockModule('../src/repositories/pokemonRepository.js', () => mockPokemonRepository);

// Import AFTER mocking
const pokemonService = await import('../src/services/pokemonService.js');

describe('Pokemon Service', () => {
  // Clear mocks before each test
  beforeEach(() => {
    jest.clearAllMocks();
  });

  // Test data
  const mockPokemonData = {
    id: 25,
    name: 'pikachu',
    sprites: {
      front_default: 'sprite.png',
      other: { 'official-artwork': { front_default: 'artwork.png' } }
    },
    types: [{ type: { name: 'electric' } }],
    height: 4,
    weight: 60,
    abilities: [
      { ability: { name: 'static' }, is_hidden: false },
      { ability: { name: 'lightning-rod' }, is_hidden: true }
    ],
    stats: [
      { stat: { name: 'hp' }, base_stat: 35 },
      { stat: { name: 'attack' }, base_stat: 55 }
    ]
  };

  const mockSpeciesData = {
    color: { name: 'yellow' },
    capture_rate: 190,
    base_happiness: 70,
    flavor_text_entries: [
      { language: { name: 'en' }, flavor_text: 'A mouse Pokemon.' }
    ],
    genera: [
      { language: { name: 'en' }, genus: 'Mouse Pokemon' }
    ]
  };

  describe('getPokemonDetails', () => {
    it('should return formatted pokemon details', async () => {
      // Arrange
      mockPokemonRepository.getPokemonByNameOrId.mockResolvedValue(mockPokemonData);
      mockPokemonRepository.getPokemonSpecies.mockResolvedValue(mockSpeciesData);

      // Act
      const result = await pokemonService.getPokemonDetails('pikachu');

      // Assert
      expect(result).toMatchObject({
        id: 25,
        name: 'pikachu',
        displayName: 'Pikachu',
        types: ['electric'],
        height: 0.4,   // Converted from decimeters
        weight: 6,     // Converted from hectograms
        color: 'yellow',
        genus: 'Mouse Pokemon'
      });
      expect(result.abilities).toHaveLength(2);
      expect(result.stats).toHaveLength(2);
    });

    it('should return null for non-existent pokemon', async () => {
      mockPokemonRepository.getPokemonByNameOrId.mockResolvedValue(null);

      const result = await pokemonService.getPokemonDetails('nonexistent');

      expect(result).toBeNull();
    });

    it('should handle missing species data gracefully', async () => {
      mockPokemonRepository.getPokemonByNameOrId.mockResolvedValue(mockPokemonData);
      mockPokemonRepository.getPokemonSpecies.mockRejectedValue(new Error('Not found'));

      const result = await pokemonService.getPokemonDetails('pikachu');

      expect(result).toBeDefined();
      expect(result.color).toBe('gray');  // Default value
      expect(result.description).toBe('No description available.');
    });
  });
});
```

## Testing API Endpoints

Create `tests/api.test.js`:

```javascript
import { jest } from '@jest/globals';
import request from 'supertest';

// Mock the service
const mockPokemonService = {
  getAllPokemon: jest.fn(),
  getPokemonDetails: jest.fn(),
  searchPokemon: jest.fn(),
  getPokemonTypes: jest.fn(),
  getPokemonByType: jest.fn()
};

jest.unstable_mockModule('../src/services/pokemonService.js', () => mockPokemonService);

// Import app after mocking
const { default: app } = await import('../src/app.js');

describe('Pokemon API Endpoints', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('GET /api/pokemon', () => {
    it('should return pokemon list with pagination', async () => {
      // Arrange
      const mockData = {
        pokemon: [
          { id: 1, name: 'bulbasaur', displayName: 'Bulbasaur', types: ['grass', 'poison'] }
        ],
        totalCount: 1000,
        currentPage: 1,
        totalPages: 50,
        hasNextPage: true,
        hasPrevPage: false
      };
      mockPokemonService.getAllPokemon.mockResolvedValue(mockData);

      // Act
      const response = await request(app).get('/api/pokemon');

      // Assert
      expect(response.status).toBe(200);
      expect(response.body.success).toBe(true);
      expect(response.body.data.pokemon).toHaveLength(1);
    });

    it('should accept page and limit query params', async () => {
      const mockData = { pokemon: [], totalCount: 1000, currentPage: 2 };
      mockPokemonService.getAllPokemon.mockResolvedValue(mockData);

      await request(app).get('/api/pokemon?page=2&limit=10');

      expect(mockPokemonService.getAllPokemon).toHaveBeenCalledWith(2, 10);
    });

    it('should return 500 on service error', async () => {
      mockPokemonService.getAllPokemon.mockRejectedValue(new Error('Service error'));

      const response = await request(app).get('/api/pokemon');

      expect(response.status).toBe(500);
      expect(response.body.success).toBe(false);
    });
  });

  describe('GET /api/pokemon/:nameOrId', () => {
    it('should return pokemon details', async () => {
      const mockPokemon = {
        id: 25,
        name: 'pikachu',
        displayName: 'Pikachu',
        types: ['electric']
      };
      mockPokemonService.getPokemonDetails.mockResolvedValue(mockPokemon);

      const response = await request(app).get('/api/pokemon/pikachu');

      expect(response.status).toBe(200);
      expect(response.body.data.name).toBe('pikachu');
    });

    it('should return 404 for non-existent pokemon', async () => {
      mockPokemonService.getPokemonDetails.mockResolvedValue(null);

      const response = await request(app).get('/api/pokemon/nonexistent');

      expect(response.status).toBe(404);
      expect(response.body.success).toBe(false);
    });
  });
});

describe('View Endpoints', () => {
  it('should render home page', async () => {
    mockPokemonService.getAllPokemon.mockResolvedValue({
      pokemon: [],
      totalCount: 0,
      currentPage: 1,
      totalPages: 1,
      hasNextPage: false,
      hasPrevPage: false
    });
    mockPokemonService.getPokemonTypes.mockResolvedValue([]);

    const response = await request(app).get('/');

    expect(response.status).toBe(200);
    expect(response.type).toBe('text/html');
  });
});
```

## Common Jest Matchers

```javascript
// Equality
expect(value).toBe(expected);           // Strict equality
expect(obj).toEqual({ a: 1 });         // Deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeDefined();

// Numbers
expect(num).toBeGreaterThan(3);
expect(num).toBeLessThan(10);

// Strings
expect(str).toMatch(/pattern/);
expect(str).toContain('substring');

// Arrays
expect(arr).toHaveLength(3);
expect(arr).toContain('item');

// Objects
expect(obj).toMatchObject({ key: 'value' });
expect(obj).toHaveProperty('key');

// Exceptions
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('error message');
```

## Mocking in Jest

### Mock Functions

```javascript
const mockFn = jest.fn();

// Configure return values
mockFn.mockReturnValue('value');        // Sync return
mockFn.mockResolvedValue('value');      // Async resolve
mockFn.mockRejectedValue(new Error());  // Async reject

// Check calls
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith('arg');
expect(mockFn).toHaveBeenCalledTimes(2);
```

### Mocking Modules

```javascript
// Mock an entire module
jest.unstable_mockModule('./module.js', () => ({
  functionA: jest.fn(),
  functionB: jest.fn()
}));

// Import AFTER mocking
const module = await import('./module.js');
```

## Running Tests

```bash
# Run all tests
npm test

# Run with watch mode (re-run on changes)
npm test -- --watch

# Run specific test file
npm test -- tests/api.test.js

# Run tests matching a pattern
npm test -- -t "getPokemonDetails"
```

## Understanding Test Output

```
 PASS  tests/pokemonService.test.js
  Pokemon Service
    getPokemonDetails
      ✓ should return formatted pokemon details (15 ms)
      ✓ should return null for non-existent pokemon (3 ms)
      ✓ should handle missing species data (5 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Coverage:    85.7%
```

## Code Coverage

Coverage shows how much of your code is tested:

| Metric | Meaning |
|--------|---------|
| **Statements** | % of statements executed |
| **Branches** | % of if/else branches covered |
| **Functions** | % of functions called |
| **Lines** | % of lines executed |

View detailed coverage report in `coverage/lcov-report/index.html`.

## Best Practices

1. **Test one thing per test** - Clear, focused tests
2. **Use descriptive names** - `it('should return null for non-existent pokemon')`
3. **Clean up after tests** - Use `beforeEach` to reset mocks
4. **Test edge cases** - Empty arrays, null values, errors
5. **Don't test implementation details** - Test behavior, not internals

## Summary

| File | Tests |
|------|-------|
| `pokemonService.test.js` | Service logic, data transformation |
| `api.test.js` | HTTP endpoints, request/response |
| `pokemonRepository.test.js` | API calls (optional) |

## Commit Your Progress

The test files are already provided in the repository. If you made any changes or additions, commit them:

```bash
git add .
git commit -m "test: verify test suite passes

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## What's Next?

Let's put it all together and run the application!

Next: [11 - Running the App](./11-running-the-app.md)
