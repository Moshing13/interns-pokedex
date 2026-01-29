# 01 - Project Setup

In this section, you'll fork the repository, clone it to your machine, and set up your development environment.

## Step 1: Fork the Repository

1. Go to the repository on GitHub
2. Click the **Fork** button in the top-right corner
3. Select your GitHub account as the destination
4. Wait for the fork to complete

You now have your own copy of the repository under your GitHub account.

## Step 2: Clone Your Fork

Open your terminal and clone your forked repository:

```bash
git clone https://github.com/YOUR-USERNAME/inter-workshop.git
cd inter-workshop
```

Replace `YOUR-USERNAME` with your actual GitHub username.

## Step 3: Create Your Branch

Create a new branch for your work. Use the format `lastname/pokedex-pull-request`:

```bash
git checkout -b dela-cruz/pokedex-pull-request
```

> **Note:** Replace `dela-cruz` with your actual last name (use lowercase and hyphens for spaces).

### Branch Naming Examples

| Your Name | Branch Name |
|-----------|-------------|
| Juan Dela Cruz | `dela-cruz/pokedex-pull-request` |
| Maria Santos | `santos/pokedex-pull-request` |
| John Smith | `smith/pokedex-pull-request` |

## Step 4: Install Dependencies

Install all the project dependencies:

```bash
npm install
```

This reads `package.json` and installs all required packages into `node_modules/`.

### What Gets Installed

The project uses these dependencies:

**Production Dependencies:**

| Package | Version | Purpose |
|---------|---------|---------|
| `express` | ^4.18.2 | Web framework for handling HTTP requests |
| `ejs` | ^3.1.9 | Template engine for rendering HTML |
| `axios` | ^1.6.2 | HTTP client for API requests |
| `dotenv` | ^16.3.1 | Load environment variables from `.env` file |

**Development Dependencies:**

| Package | Version | Purpose |
|---------|---------|---------|
| `nodemon` | ^3.0.2 | Auto-restart server on file changes |
| `jest` | ^29.7.0 | Testing framework |
| `supertest` | ^6.3.3 | HTTP testing library |
| `eslint` | ^9.17.0 | Code linting (find errors/issues) |
| `prettier` | ^3.7.4 | Code formatting |

## Step 5: Set Up Environment Variables

Copy the example environment file:

```bash
cp .env.example .env
```

On Windows (Command Prompt):
```cmd
copy .env.example .env
```

On Windows (PowerShell):
```powershell
Copy-Item .env.example .env
```

The `.env` file contains:

```env
PORT=3000
NODE_ENV=development
POKEAPI_BASE_URL=https://pokeapi.co/api/v2
DEFAULT_PAGE_LIMIT=20
MAX_SEARCH_LIMIT=1000
```

## Step 6: Create Project Directories

Create the directories you'll need for your code:

```bash
mkdir -p src/config src/routes src/controllers src/services src/repositories src/views/partials public/css
```

On Windows (Command Prompt):
```cmd
mkdir src\config src\routes src\controllers src\services src\repositories src\views\partials public\css
```

Your project should now look like this:

```
inter-workshop/
├── .github/              # CI/CD workflow (pre-configured)
├── docs/                 # Documentation
│   └── guide/            # This tutorial
├── public/               # Static assets (you created this)
│   └── css/
├── src/                  # Your application code (you created this)
│   ├── config/
│   ├── controllers/
│   ├── repositories/
│   ├── routes/
│   ├── services/
│   └── views/
│       └── partials/
├── tests/                # Test files (pre-configured)
├── .env                  # Your environment variables
├── .env.example          # Environment template
├── .gitignore
├── .prettierrc
├── eslint.config.js
├── jest.config.js
├── package.json
└── package-lock.json
```

## Step 7: Verify Setup

Make sure everything is set up correctly:

```bash
# Check Node.js
node --version

# Check npm packages are installed
npm list --depth=0

# Check you're on your branch
git branch
```

You should see your branch name highlighted (e.g., `* dela-cruz/pokedex-pull-request`).

## Available npm Scripts

The `package.json` is already configured with these scripts:

| Script | Command | Description |
|--------|---------|-------------|
| `start` | `npm start` | Run the app in production |
| `dev` | `npm run dev` | Run with auto-reload (development) |
| `test` | `npm test` | Run tests with coverage report |
| `lint` | `npm run lint` | Check code for errors |
| `lint:fix` | `npm run lint:fix` | Auto-fix linting errors |
| `format` | `npm run format` | Format code with Prettier |
| `format:check` | `npm run format:check` | Check if code is formatted |

> **Note:** You don't need to modify `package.json`. Everything is already set up for you.

## Commit Your Progress

Now that your project is set up, commit your changes:

```bash
git add .
git commit -m "chore: set up project structure and environment

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## What's Next?

Your development environment is ready! In the next section, we'll learn about the architecture pattern we're using.

Next: [02 - Understanding Architecture](./02-understanding-architecture.md)
