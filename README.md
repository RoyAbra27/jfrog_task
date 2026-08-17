# User Management Service

A simple Node.js API service for managing users. This service provides basic CRUD operations for user management with REST endpoints.

## Security analysis (assignment fork - start here)

This fork adds a full Xray security pass over the service; the deliverables live in [docs/](docs/):

- [FINDINGS.md](docs/FINDINGS.md) - baseline scan results and the 2 applicable app-level High CVEs, with evidence
- [CONTEXTUAL-ANALYSIS.md](docs/CONTEXTUAL-ANALYSIS.md) - what contextual analysis is and how it reshaped prioritization
- [REMEDIATION.md](docs/REMEDIATION.md) - the fix (moment CVE-2022-31129), verified gone on rescan
- [BUILD-PUBLISH.md](docs/BUILD-PUBLISH.md) - the OIDC-authenticated build/publish workflow and its green run

## Features

- User CRUD operations (Create, Read, Update, Delete)
- User search and pagination
- JWT-based authentication
- Input validation
- External API integration
- Random user generation for testing
- Docker support

## Tech Stack

- Node.js
- Express.js
- JWT for authentication
- bcrypt for password hashing
- Lodash for utility functions
- Moment.js for date handling
- Axios for HTTP requests
- Validator for input validation

## API Endpoints

### Health Check
- `GET /health` - Check service health

### Authentication
- `POST /api/auth/login` - User login

### Users
- `GET /api/users` - Get all users (with pagination and search)
- `GET /api/users/:id` - Get user by ID
- `POST /api/users` - Create new user
- `PUT /api/users/:id` - Update user
- `DELETE /api/users/:id` - Delete user
- `POST /api/users/generate-random` - Generate random test users

### External Data
- `GET /api/external-data` - Fetch and process external user data

## Quick Start

### Local Development

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the development server:
   ```bash
   npm run dev
   ```

3. The service will be available at `http://localhost:3000`

### Production

1. Install dependencies:
   ```bash
   npm install --production
   ```

2. Start the server:
   ```bash
   npm start
   ```

### Docker

1. Build the Docker image:
   ```bash
   npm run docker:build
   ```

2. Run the container:
   ```bash
   npm run docker:run
   ```

### Publishing

1. Create a package (.tgz file):
   ```bash
   npm run pack
   ```

2. Publish to npm registry:
   ```bash
   npm run publish
   ```

3. Publish to custom registry (e.g., Artifactory):
   ```bash
   # First configure your registry
   npm config set registry <your-artifactory-url>
   
   # Then publish
   npm run publish
   ```

4. Or use the helper command for Artifactory:
   ```bash
   npm run publish:artifactory
   ```

## Environment Variables

Create a `.env` file based on `.env.example`:

- `PORT` - Server port (default: 3000)
- `NODE_ENV` - Environment (development/production)
- `JWT_SECRET` - Secret key for JWT tokens

## Testing

Run tests:
```bash
npm test
```

## API Examples

### Create a user
```bash
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"username": "newuser", "email": "user@example.com"}'
```

### Get all users
```bash
curl http://localhost:3000/api/users
```

### Login
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "password123"}'
```

## License

MIT 