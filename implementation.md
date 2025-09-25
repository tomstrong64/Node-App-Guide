# Feature Implementation Guide

This guide consolidates all standards and practices for implementing features in Node.js applications. Follow these rules when building new functionality.

---

## Core Programming Principles

### Fail Fast
- **ALWAYS** validate inputs and throw errors immediately on invalid state
- **NEVER** silently patch invalid data or return default values that hide problems
- **VALIDATE** at function boundaries and internal logic
- **AVOID** returning default values that mask data integrity issues

```js
// Fail fast with clear error messages - detect problems immediately
function calculateOrderTotal(order) {
  // Validate structure first
  if (!order) throw new Error("Order is required");
  if (!Array.isArray(order.items)) throw new Error("Order must have items array");
  if (order.items.length === 0) throw new Error("Order cannot be empty");
  
  // Use functional approach instead of mutable variable
  return order.items.reduce((total, item) => {
    // Validate each item's data integrity
    if (!item.price || item.price < 0) {
      throw new Error(`Invalid price for item: ${item.name || 'unknown'}`);
    }
    if (!item.quantity || item.quantity <= 0) {
      throw new Error(`Invalid quantity for item: ${item.name || 'unknown'}`);
    }
    return total + (item.price * item.quantity);
  }, 0);
}
```

### Return Early
- **ALWAYS** return immediately when a result is known
- **AVOID** mutable variables that accumulate final return values
- **USE** guard clauses to reduce nesting
- **PREFER** multiple returns over complex conditional logic

```js
// Return early with guard clauses - eliminates nested conditions
function calculateShippingCost(order, user) {
  // Handle edge cases first with early returns
  if (!order) return 0;
  if (!order.items || order.items.length === 0) return 0;
  
  const totalWeight = order.items.reduce((sum, item) => sum + item.weight, 0);
  
  // Business rules with early returns
  if (user.isPremium) return 0; // Free shipping for premium users
  if (totalWeight > 50) return 25; // Heavy items
  if (order.total > 100) return 0; // Free shipping over $100
  
  return 10; // Standard shipping
}
```

### Never Nester (Max 3 Levels)
- **LIMIT** indentation to maximum 3 levels: Function → Conditional → Loop
- **USE** guard clauses (`return`, `continue`, `throw`) to flatten code
- **EXTRACT** smaller functions if more nesting is needed
- **AVOID** deeply nested conditional logic

```js
// Flat structure with guard clauses - max 3 indentation levels
function processUserOrders(users) {
  if (!users?.length) return [];
  
  const processedOrders = [];
  
  for (const user of users) {
    // Use continue to skip invalid users
    if (!user.isActive) continue;
    if (!user.orders?.length) continue;
    
    for (const order of user.orders) {
      // Use continue to skip invalid orders
      if (order.status !== 'pending') continue;
      if (!isValidOrder(order)) continue;
      
      processedOrders.push(processOrder(order));
    }
  }
  
  return processedOrders;
}
```

### Single Responsibility Principle
- **ONE** reason to change per module/class/function
- **SEPARATE** concerns: Controllers handle HTTP, Services handle business logic, Repositories handle data
- **AVOID** mixing HTTP handling, business logic, and data access in the same function
- **USE** named exports and dependency injection

```js
// Controller - only handles HTTP concerns
// src/controllers/userController.js
import { matchedData } from 'express-validator';
import { catchNext } from '../utils/catchNext.js';
import * as userService from '../services/userService.js';

export const createUser = catchNext(async (req, res, next) => {
  // Use matchedData instead of req.body
  const userData = matchedData(req, { locations: ['body'] });
  const user = await userService.createUser(userData);
  res.status(201).json(user);
});
```

// Service - handles business logic
// src/services/userService.js
import * as userRepo from '../repositories/userRepository.js';
import * as emailService from './emailService.js';

export async function createUser(userData) {
  const user = await userRepo.createUser(userData);
  await emailService.sendWelcomeEmail(user.email);
  return user;
}

// Repository - handles data access
// src/repositories/userRepository.js
import * as User from '../models/User.js';

export async function createUser(userData) {
  return await User.create(userData);
}
```

### Composition Over Inheritance
- **PREFER** "has-a" relationships over "is-a"
- **INJECT** dependencies through constructor parameters
- **AVOID** deep inheritance hierarchies
- **USE** dependency injection for loose coupling

```js
// Composition - inject dependencies for flexible, testable code
class Logger {
  log(message) { console.log(`[${new Date().toISOString()}] ${message}`); }
}

class EmailService {
  constructor(logger) {
    this.logger = logger; // Has-a relationship with logger
  }
  
  async sendEmail(to, subject, body) {
    this.logger.log(`Sending email to ${to}`);
    // Email sending logic
  }
}

class UserService {
  constructor(userRepository, emailService, logger) {
    this.userRepository = userRepository; // Has-a relationship
    this.emailService = emailService;     // Has-a relationship
    this.logger = logger;                 // Has-a relationship
  }
  
  async createUser(userData) {
    this.logger.log("Creating user");
    const user = await this.userRepository.create(userData);
    await this.emailService.sendEmail(user.email, "Welcome", "Welcome to our app!");
    return user;
  }
}
```

---

## Node.js Standards

### Package Configuration
- **ALWAYS** use `"type": "module"` in package.json for ESM
- **PREFER** native Node.js features over packages when available
- **STORE** tool configurations in package.json when supported
- **NO** `dev` or `start` scripts - application runs in Docker only

```json
{
  "type": "module",
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --check .",
    "format:fix": "prettier --write .",
    "test": "vitest",
    "test:watch": "vitest --watch"
  }
}
```

### Environment Configuration
- **CENTRALIZE** all environment variables in a config module
- **USE** nullish coalescing (`??`) for default values
- **NEVER** access `process.env` directly in application code

```js
// src/config/index.js
export const config = {
  port: process.env.PORT ?? 3000,
  dbUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET
};
```

---

## Project Structure

### Required Folder Structure
```
/src/
├── bin/             # Entrypoints (index.js)
├── config/          # Centralized configuration
├── constants/       # Application-wide constants
├── controllers/     # Request controllers
├── errors/          # Custom error classes/handlers
├── middleware/      # Express middlewares
├── models/          # Database models
├── repositories/    # Data access layer
├── routes/          # Express route definitions
├── services/        # Business logic
├── utils/           # Utility/helper functions
└── app.js           # App bootstrap
```

### Structure Rules
- **ALL** source code lives in `/src`
- **ENTRYPOINT** is `/src/bin/index.js`
- **USE** plural folder names (`controllers`, `services`, etc.)
- **SEPARATE** concerns: controllers, services, repositories
- **PLACE** tests in `/test` directory

---

## Express Validation and Data Handling

### Request Validation
- **ALWAYS** use express-validator for validation and throwing 400 errors
- **ALWAYS** use `matchedData(req, { locations: ['body'] })` from express-validator for getting the request body
- **NEVER** access `req.body` directly - use `matchedData(req, { locations: ['body'] })` instead
- **SEPARATE** validation into middleware, not in controllers
- **USE** validation chains with `handleValidationErrors` middleware
- **WRAP** all async middleware and controllers with `catchNext` utility

```js
// src/middleware/validation.js
import { validationResult } from 'express-validator';
import { catchNext } from '../utils/catchNext.js';

export const handleValidationErrors = catchNext((req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  next();
});

// src/middleware/userValidation.js
import { body } from 'express-validator';
import { handleValidationErrors } from './validation.js';

export const validateCreateUser = [
  body('email').isEmail().withMessage('Valid email is required.'),
  body('firstName').trim().notEmpty().withMessage('First name is required.'),
  body('lastName').trim().notEmpty().withMessage('Last name is required.'),
  handleValidationErrors,
];

// src/controllers/userController.js
import { matchedData } from 'express-validator';
import { catchNext } from '../utils/catchNext.js';
import * as userService from '../services/userService.js';

export const createUser = catchNext(async (req, res, next) => {
  // Use matchedData instead of req.body
  const userData = matchedData(req, { locations: ['body'] });
  const user = await userService.createUser(userData);
  res.status(201).json(user);
});
```

### Repository Patterns
- **BE EXPLICIT** about data used in repository functions
- **CREATE** indexes on mongoose models for any get methods
- **SPECIFY** exactly what you're querying by in get methods

```js
// src/models/User.js - with indexes
import mongoose from 'mongoose';

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  isActive: { type: Boolean, default: true }
});

// Create indexes for repository get methods
userSchema.index({ email: 1 }); // For getUserByEmail
userSchema.index({ isActive: 1 }); // For getActiveUsers

export default mongoose.model('User', userSchema);
```

```js
// src/repositories/userRepository.js
import * as User from '../models/User.js';

// Explicit about what data is used
export async function getUserByEmail(email) {
  return await User.findOne({ email });
}

export async function getUserById(userId) {
  return await User.findById(userId);
}

export async function getActiveUsers() {
  return await User.find({ isActive: true });
}

// Create methods are less explicit due to schema validation
export async function createUser(userData) {
  return await User.create(userData);
}
```

### Resource Loading and Route Parameters
- **LOAD** resources at the earliest possible time in HTTP requests
- **ADD** loaded resources to the request object
- **PASS** resources down to functions that need them
- **NEVER** use generic `id` parameter names - be specific

```js
// src/routes/userRoutes.js
import { getUserById } from '../repositories/userRepository.js';

// Specific parameter names, not just 'id'
app.get('/users/:userId', loadUser, getUser);
app.get('/users/:userId/orders/:orderId', loadUser, loadOrder, getUserOrder);

// Middleware to load resources early
import { catchNext } from '../utils/catchNext.js';

const loadUser = catchNext(async (req, res, next) => {
  const user = await getUserById(req.params.userId);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  req.user = user; // Add to request object
  next();
});

const loadOrder = catchNext(async (req, res, next) => {
  const order = await getOrderById(req.params.orderId);
  if (!order) {
    return res.status(404).json({ error: 'Order not found' });
  }
  req.order = order; // Add to request object
  next();
});

// Controller uses pre-loaded resources
export const getUser = catchNext(async (req, res) => {
  // req.user already loaded by middleware
  res.json(req.user);
});

export const getUserOrder = catchNext(async (req, res) => {
  // Both req.user and req.order already loaded
  const order = await getOrderWithUserDetails(req.order, req.user);
  res.json(order);
});
```

---

## HTTP Request Flow Standards

### Request Processing Order
**FOLLOW** this exact sequence for all API endpoints:

1. **Route exists?** → No → `404 Not Found`
2. **Route has resource?** → No → pass to validation or controller
3. **User logged in?** → No → `401 Unauthorized`
4. **Resource exists?** → No → `404 Not Found`
5. **User has access to resource?** → No → `404 Not Found` (hide existence)
6. **User has access to route?** → No → `403 Forbidden`
7. **Request content valid?** → No → `400 Bad Request`
8. **Otherwise** → proceed to controller

### Middleware Order Example
```js
// src/routes/userRoutes.js
import { validateCreateUser, validateUpdateUser } from '../middleware/userValidation.js';
import { authenticate, anonymous } from '../middleware/auth.js';
import { populate, authoriseResource, authoriseRoute } from '../middleware/authorization.js';
import { Resources, Access } from '../constants/resources.js';
import { createUser, getUser, updateUser, deleteUser } from '../controllers/userController.js';

// POST /users - Create user (no auth required)
router.post(
  '/',
  anonymous,
  validateCreateUser,
  createUser,
);

// Apply shared middleware to all routes with :userId parameter
router.use('/:userId', authenticate);

router.use(
  '/:userId',
  populate(Resources.USER),
  authoriseResource(Resources.USER),
);

// GET /users/:userId - Get user (auth + resource access required)
router.get('/:userId', getUser);

// PUT /users/:userId - Update user (auth + resource + route permissions required)
router.put(
  '/:userId',
  authoriseRoute(Access.ADMIN),
  validateUpdateUser,
  updateUser,
);

// DELETE /users/:userId - Delete user (auth + resource + route permissions required)
router.delete(
  '/:userId',
  authoriseRoute(Access.ADMIN),
  deleteUser,
);
```

### Status Code Rules
- **ALWAYS** return the most restrictive applicable status code early
- **PREFER** `404` for hidden resources (don't expose existence)
- **DISTINGUISH** between:
  - `401 Unauthorized` → not logged in
  - `403 Forbidden` → logged in but not allowed
  - `400 Bad Request` → validation errors or malformed input

---

## Package Selection

### HTTP Server Stack
- **express** → Web server framework
- **helmet** → Security headers middleware
- **cors** → CORS handling
- **cookie-parser** → Cookie parsing
- **multer** → File uploads
- **express-validator** → Request validation
- **morgan** → HTTP request logging

### Database
- **mongoose** → MongoDB ODM
- **mysql** → MySQL client
- **pg** → PostgreSQL client

### Messaging / Queues
- **ioredis** → Redis client with cluster + sentinel support
- **amqplib** → RabbitMQ client for Node.js

### WebSockets
- **socket.io** → Real-time bidirectional communication
- **@socket.io/redis-streams-adapter** → Redis Streams adapter for horizontal scaling

### Testing
- **vitest** → Unit + integration testing
- **supertest** → HTTP API testing

### Documentation
- **swagger-jsdoc** → Swagger/OpenAPI generation from comments
- **swagger-ui-express** → Swagger UI frontend
- **mongoose-to-swagger** → Generate Swagger schemas from mongoose models

### Authentication
- **jsonwebtoken** → JWT handling
- **bcrypt** → Password hashing
- **passport** → Authentication strategies

### Styling / Linting
- **eslint** → Linting
- **eslint-config-airbnb-base** → Airbnb rules
- **eslint-config-prettier** → Disable ESLint rules conflicting with Prettier
- **eslint-plugin-import** → Import/export rules
- **prettier** → Formatting

### Utilities
- **dayjs** → Modern date/time API
- **compression** → Gzip/deflate middleware

### Package Selection Rules
- **PRIORITIZE** stability and low dependency count
- **AVOID** deprecated packages
- **USE** native Node.js features instead of packages when possible
- **CHECK** if Node.js provides native solution before adding package

---

## Implementation Checklist

### Before Starting
- [ ] Review folder structure and place files correctly
- [ ] Check if native Node.js features can replace packages
- [ ] Plan HTTP flow and status codes
- [ ] Identify required packages from approved list

### During Implementation
- [ ] Follow fail-fast validation patterns
- [ ] Use return early to reduce nesting
- [ ] Keep indentation to max 3 levels
- [ ] Apply single responsibility principle
- [ ] Use composition over inheritance
- [ ] Centralize environment variables in config
- [ ] Follow HTTP request flow sequence
- [ ] Use appropriate status codes
- [ ] Use express-validator for all validation
- [ ] Use matchedData() instead of req.body
- [ ] Create indexes for repository get methods
- [ ] Be explicit about data used in repository functions
- [ ] Load resources early and add to request object
- [ ] Use specific parameter names (not just 'id')

### After Implementation
- [ ] Verify all files are in correct directories
- [ ] Check that no deprecated packages were used
- [ ] Ensure proper error handling and validation
- [ ] Confirm HTTP flow follows the standard sequence

