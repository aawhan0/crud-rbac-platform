# Auto-Generated CRUD + RBAC Platform

A low-code internal developer platform that allows users to define data models via a web UI and automatically generates REST APIs, admin interfaces, and role-based access control.

## 🎯 Overview

This platform enables teams to:
- **Define Models** through a simple web form (no coding required)
- **Auto-Generate APIs** - CRUD endpoints created instantly
- **Enforce RBAC** - Role-based permissions (Admin, Manager, Viewer) applied automatically
- **Manage Data** - Generic admin UI that works with any model
- **Persist Models** - Model definitions saved as JSON files for versioning and reproducibility

### Key Features
✅ Dynamic model definition via web UI  
✅ Auto-generated REST APIs (POST, GET, PUT, DELETE)  
✅ File-based model persistence (`/models` directory)  
✅ Role-based access control with permission enforcement  
✅ Ownership-based authorization (users can only modify their own records)  
✅ Dynamic admin interface for data management  
✅ JWT authentication  
✅ Extensible architecture for custom validators and hooks  

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Admin UI (React/Vite)                   │
│  - Model Editor (Create/Edit/Publish Models)             │
│  - Data Management (CRUD for any model)                  │
│  - Dashboard & Reporting                                 │
└────────────────┬────────────────────────────────────────┘
                 │ HTTP/REST
┌────────────────▼────────────────────────────────────────┐
│              Express Backend (TypeScript)                │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Authentication & JWT Middleware                │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  RBAC Permission Checking Middleware            │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  Dynamic Route Registration                     │   │
│  │  (CRUD endpoints generated per model)           │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  Model Management API                           │   │
│  │  (Save, Load, Publish models)                   │   │
│  └──────────────────────────────────────────────────┘   │
└────────────┬──────────────────────────────────────────┬─┘
             │                                          │
    File I/O │                                          │ Database
             │                                          │
    ┌────────▼──────────┐                    ┌──────────▼────────┐
    │  /models Directory │                    │  PostgreSQL/SQLite│
    │  (JSON Model Defs) │                    │  (Application Data)│
    └───────────────────┘                    └───────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites
- Node.js 16+ and npm
- PostgreSQL 12+ (or use SQLite for development)
- Git

### Installation

#### 1. Clone & Install Dependencies
```bash
git clone https://github.com/yourusername/crud-rbac-platform.git
cd crud-rbac-platform

# Backend
cd backend
npm install

# Frontend
cd ../frontend
npm install
```

#### 2. Set Up Environment Variables

**Backend** (`backend/.env`):
```env
DATABASE_URL="postgresql://user:password@localhost:5432/crud_platform"
# Or for SQLite: "file:./dev.db"

JWT_SECRET="your-super-secret-jwt-key-change-this"
PORT=5000
NODE_ENV=development
```

**Frontend** (`frontend/.env.local`):
```env
VITE_API_BASE_URL=http://localhost:5000/api
```

#### 3. Initialize Database

```bash
cd backend

# Generate Prisma client
npm run prisma:generate

# Run migrations
npm run prisma:migrate

# Seed initial roles and admin user
npm run prisma:seed
```

#### 4. Start the Application

**Terminal 1 - Backend**:
```bash
cd backend
npm run dev
# Server running on http://localhost:5000
```

**Terminal 2 - Frontend**:
```bash
cd frontend
npm run dev
# UI running on http://localhost:5173
```

#### 5. Login to Admin Dashboard
- URL: `http://localhost:5173`
- Default Admin Credentials:
  - Email: `admin@example.com`
  - Password: `admin123`

---

## 📖 How to Use

### Creating & Publishing a Model

1. **Navigate to Model Editor**
   - Go to Admin Dashboard → "Model Editor"

2. **Define Your Model**
   - Model Name: `Product` (required)
   - Table Name: `products` (auto-filled, editable)
   - Add Fields:
     - `name` (type: string, required: true)
     - `price` (type: number, required: true)
     - `description` (type: string, required: false)
     - `ownerId` (type: string, owner field: true)

3. **Set RBAC Permissions**
   - Admin: `["all"]` (all operations)
   - Manager: `["create", "read", "update"]`
   - Viewer: `["read"]`

4. **Click "Publish"**
   - Model JSON saved to `/backend/models/Product.json`
   - CRUD endpoints registered at `/api/products`
   - Admin UI automatically adds Product data management section

### Example Model Definition (JSON)

File: `/backend/models/Product.json`
```json
{
  "id": "product",
  "name": "Product",
  "tableName": "products",
  "fields": [
    {
      "name": "name",
      "type": "string",
      "required": true,
      "unique": false
    },
    {
      "name": "price",
      "type": "number",
      "required": true,
      "default": 0
    },
    {
      "name": "description",
      "type": "string",
      "required": false
    },
    {
      "name": "ownerId",
      "type": "string",
      "required": true,
      "isOwnerField": true
    }
  ],
  "ownerField": "ownerId",
  "rbac": {
    "Admin": ["all"],
    "Manager": ["create", "read", "update"],
    "Viewer": ["read"]
  },
  "createdAt": "2025-11-30T12:00:00Z",
  "updatedAt": "2025-11-30T12:00:00Z"
}
```

---

## 🔌 API Endpoints

### Authentication
```
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/register (admin only)
GET    /api/auth/me
```

### Model Management
```
POST   /api/models/publish          # Publish a new model
GET    /api/models                  # List all published models
GET    /api/models/:id              # Get model details
PUT    /api/models/:id              # Update model definition
DELETE /api/models/:id              # Delete model
```

### Dynamic CRUD (per model)
```
POST   /api/<modelName>             # Create record
GET    /api/<modelName>             # Read all records (paginated)
GET    /api/<modelName>/:id         # Read single record
PUT    /api/<modelName>/:id         # Update record (owner/admin only)
DELETE /api/<modelName>/:id         # Delete record (owner/admin only)
```

### Example: Product CRUD
```bash
# Create a product
curl -X POST http://localhost:5000/api/products \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Laptop",
    "price": 999.99,
    "description": "High-performance laptop"
  }'

# Get all products
curl -X GET http://localhost:5000/api/products \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# Update product (owner/admin only)
curl -X PUT http://localhost:5000/api/products/1 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"price": 899.99}'

# Delete product (owner/admin only)
curl -X DELETE http://localhost:5000/api/products/1 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

## 🔐 How RBAC Works

### Role Definitions
- **Admin**: Can perform all operations on all models (create, read, update, delete)
- **Manager**: Can create, read, update records; cannot delete
- **Viewer**: Can only read records

### Permission Enforcement

1. **Operation-Level Check**: User's role must have permission for the CRUD operation
2. **Ownership Check**: For update/delete, if model has `ownerField`:
   - User must be the record owner OR be Admin
   - Non-owners get 403 Forbidden

### Middleware Flow
```
Request arrives with JWT
  ↓
Extract user & role from JWT
  ↓
Check if role has permission for operation
  ↓
(For update/delete) Check ownership if applicable
  ↓
Permission granted → Process request
Permission denied → Return 403 Forbidden
```

### Example: RBAC in Action

Model: **Employee** with `ownerId` field
```
Admin User → Can create, read, update, delete ANY employee ✅
Manager User → Can create, read, update ANY employee, but cannot delete ✅
Manager User → Cannot update Employee owned by another Manager ❌
Viewer User → Can only read employees ✅
```

---

## 📁 Project Structure

```
crud-rbac-platform/
├── backend/
│   ├── src/
│   │   ├── index.ts                 # Express server entry
│   │   ├── middleware/
│   │   │   ├── auth.ts              # JWT verification
│   │   │   └── rbac.ts              # Permission checking
│   │   ├── routes/
│   │   │   ├── auth.ts              # Authentication routes
│   │   │   ├── models.ts            # Model management routes
│   │   │   └── dynamic.ts           # Dynamic CRUD routes
│   │   ├── controllers/
│   │   │   ├── auth.controller.ts
│   │   │   ├── models.controller.ts
│   │   │   └── crud.controller.ts
│   │   ├── services/
│   │   │   ├── auth.service.ts
│   │   │   ├── model.service.ts     # Model file I/O
│   │   │   └── rbac.service.ts
│   │   └── utils/
│   │       ├── validators.ts
│   │       └── helpers.ts
│   ├── models/                      # Published model definitions (JSON)
│   │   ├── Product.json
│   │   └── Employee.json
│   ├── prisma/
│   │   ├── schema.prisma            # Database schema
│   │   ├── seed.ts                  # Initial data seed
│   │   └── migrations/
│   ├── .env                         # Environment variables
│   ├── tsconfig.json
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ModelEditor.tsx      # Model definition form
│   │   │   ├── DynamicForm.tsx      # Generic form renderer
│   │   │   ├── DynamicTable.tsx     # Generic table renderer
│   │   │   └── ModelList.tsx        # Models list
│   │   ├── pages/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── ModelEditorPage.tsx
│   │   │   ├── DataManagementPage.tsx
│   │   │   └── LoginPage.tsx
│   │   ├── hooks/
│   │   │   ├── useAuth.ts           # Auth context hook
│   │   │   └── useApi.ts            # API calls hook
│   │   ├── contexts/
│   │   │   └── AuthContext.tsx
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── .env.local
│   ├── vite.config.ts
│   └── package.json
│
├── README.md                        # This file
├── .gitignore
└── docker-compose.yml               # (Optional) Docker setup
```

---

## 🧪 Testing

### Run Backend Tests
```bash
cd backend
npm run test
```

### Run Specific Test Suite
```bash
npm run test -- auth.test.ts        # Auth tests
npm run test -- rbac.test.ts        # RBAC tests
npm run test -- crud.test.ts        # CRUD tests
```

### Test Coverage
```bash
npm run test:coverage
```

### Manual API Testing with Postman/Curl

See **API Endpoints** section above for example curl commands.

---

## 🔄 File-Based Model Persistence

### How It Works

**Publishing a Model** (from Admin UI):
1. User fills form → sends POST to `/api/models/publish`
2. Backend validates model definition
3. Backend writes JSON to `/backend/models/<ModelName>.json`
4. Backend loads model schema and registers CRUD routes dynamically
5. Admin UI updates to show new model

**Loading Models on Startup**:
1. Server starts → reads all `.json` files from `/backend/models/`
2. For each model, registers dynamic CRUD routes
3. Model definitions available in-memory for RBAC checks

**Example**:
```
User publishes "Product" model
↓
/backend/models/Product.json created
↓
Routes registered: POST/GET/PUT/DELETE /api/products
↓
Admin UI loads models list → shows "Product"
↓
Next server restart loads Product.json automatically
```

### Benefits
✅ Models are version-controllable (commit to git)  
✅ Easy backup (copy `/models` folder)  
✅ Schema as code (inspect/modify manually if needed)  
✅ Hot-reload capable (watch for file changes)  

---

## 🚀 Deployment

### Deploy Backend (Render/Railway/Heroku)

1. **Set environment variables** on platform:
   ```
   DATABASE_URL=<your-postgres-url>
   JWT_SECRET=<generate-a-strong-secret>
   NODE_ENV=production
   ```

2. **Build command**:
   ```
   npm run build
   ```

3. **Start command**:
   ```
   npm run start
   ```

### Deploy Frontend (Vercel/Netlify)

1. Connect GitHub repo to Vercel/Netlify
2. Set environment variable:
   ```
   VITE_API_BASE_URL=<your-backend-url>
   ```
3. Build command: `npm run build`
4. Deploy!

### Docker (Optional)

```bash
docker-compose up
```

Ensure `docker-compose.yml` and `Dockerfile` are configured.

---

## 📋 Sample Models

### Product Model
```json
{
  "name": "Product",
  "tableName": "products",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "price", "type": "number", "required": true },
    { "name": "ownerId", "type": "string", "isOwnerField": true }
  ],
  "ownerField": "ownerId",
  "rbac": {
    "Admin": ["all"],
    "Manager": ["create", "read", "update"],
    "Viewer": ["read"]
  }
}
```

### Employee Model
```json
{
  "name": "Employee",
  "tableName": "employees",
  "fields": [
    { "name": "firstName", "type": "string", "required": true },
    { "name": "lastName", "type": "string", "required": true },
    { "name": "email", "type": "string", "required": true, "unique": true },
    { "name": "department", "type": "string", "required": true },
    { "name": "salary", "type": "number", "required": false },
    { "name": "managerId", "type": "string", "required": false }
  ],
  "ownerField": "managerId",
  "rbac": {
    "Admin": ["all"],
    "Manager": ["create", "read", "update"],
    "Viewer": ["read"]
  }
}
```

---

## 🛠️ Troubleshooting

### Backend won't start
- Check `.env` file exists and `DATABASE_URL` is correct
- Ensure PostgreSQL is running (or SQLite file is accessible)
- Run `npm run prisma:migrate` to apply migrations
- Check port 5000 is not in use

### CORS errors in frontend
- Ensure backend is running on `http://localhost:5000`
- Check `VITE_API_BASE_URL` in frontend `.env.local`
- Verify backend has CORS enabled in Express config

### JWT token errors
- Generate new token: Login again
- Check `JWT_SECRET` is same in `.env`
- Token might be expired: Re-login

### Model not appearing in admin UI
- Ensure `npm run dev` is running backend
- Publish model again
- Check browser console for API errors
- Verify `/backend/models/` directory contains JSON file

---

## 🎓 Learning Resources

- [Express Documentation](https://expressjs.com/)
- [Prisma ORM Guide](https://www.prisma.io/docs/)
- [JWT Authentication](https://jwt.io/introduction)
- [React Hooks Documentation](https://react.dev/reference/react/hooks)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repo
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

---

## 📧 Questions?

Open an issue on GitHub or reach out to the maintainers.

---

## 🎯 Roadmap

- [x] Core architecture design
- [ ] MVP implementation (Phases 1-6)
- [ ] Comprehensive testing suite
- [ ] Hot-reload for models
- [ ] Field relations and foreign keys
- [ ] Audit logging
- [ ] Model versioning
- [ ] Database migration auto-generation
- [ ] Advanced filtering and search
- [ ] API rate limiting and security hardening

---

**Happy Building! 🚀**
