# Merchant Store (Medusa v2 Monorepo)

This repository contains a full-stack ecommerce solution powered by [Medusa v2](https://medusajs.com/) (Backend) and [Next.js](https://nextjs.org/) (Storefront). It is structured as an npm monorepo to easily manage both applications together.

## 🚀 Getting Started

Follow these steps to set up the project locally on your machine.

### 1. Prerequisites
- **Node.js**: v20 or newer
- **npm**: v10 or newer (included with Node.js)
- **Database**: A PostgreSQL database (e.g., [Supabase](https://supabase.com) or a local Postgres instance)

### 2. Install Dependencies
Run the following command in the root of the repository to install all dependencies for both the backend and storefront:

```bash
npm install
npm install --prefix storefront
```
*(Note: The storefront dependencies are installed separately to prevent workspace dependency hoisting conflicts with the Medusa backend).*

### 3. Environment Variables Setup

You need to configure the environment variables for both the backend and the storefront.

#### Backend
1. Navigate to the backend folder:
   ```bash
   cd backend/apps/backend
   ```
2. Copy the template to create your `.env` file:
   ```bash
   cp .env.template .env
   ```
3. Open the new `.env` file and fill in your database connection string:
   ```env
   DATABASE_URL="postgresql://postgres.YOUR_PROJECT:YOUR_PASSWORD@aws-0-REGION.pooler.supabase.com:5432/postgres"
   ```

#### Storefront
1. Navigate to the storefront folder:
   ```bash
   cd ../../../storefront
   ```
2. Copy the local template (if available) or create a new `.env.local` file.
3. Ensure it contains at least the following (you will fill in the publishable key in step 5):
   ```env
   MEDUSA_BACKEND_URL=http://localhost:9000
   NEXT_PUBLIC_BASE_URL=http://localhost:8000
   NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_... 
   ```

### 4. Database Migrations & Seeding
Before you can run the backend, you must initialize your empty database with Medusa's tables and seed it with a default admin user.

Run the following commands from the **`backend/apps/backend`** directory:

```bash
# Build the database tables and insert default regions/products
npx medusa db:migrate

# Create a default admin user (Email: admin@admin.com, Password: admin)
npx medusa user -e admin@admin.com -p admin
```

### 5. Link the Storefront
When you ran the migration command above, Medusa generated a secure **Publishable API Key** inside your database. Your Next.js storefront needs this key to fetch data.

1. Start the backend server (from the root of the project):
   ```bash
   npm run dev:backend
   ```
2. Once the server says `Server is ready on port: 9000`, open your browser and go to the Admin Dashboard: [http://localhost:9000/app](http://localhost:9000/app)
3. Log in with `admin@admin.com` and `admin`.
4. Go to **Settings > API Keys > Publishable API Keys**.
5. Copy your key (starts with `pk_...`) and paste it into your `storefront/.env.local` file as `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY`.

### 6. Run the Project
Now you can run both servers simultaneously from the root of the project. Open two separate terminal windows and run:

**Terminal 1 (Backend):**
```bash
npm run dev:backend
```

**Terminal 2 (Storefront):**
```bash
npm run dev:storefront
```

Your storefront will now be accessible at [http://localhost:8000](http://localhost:8000).

## 🛠️ Deployment (CI/CD)

This project is fully configured for Continuous Deployment on **Render**. 
When you push to the `main` branch, Render will automatically detect the `render.yaml` Blueprint file in the root directory and deploy both the backend and storefront independently.

If you are setting this up for the first time on Render, please refer to the `render_deployment_guide.md` (or your internal team notes) on how to securely inject your environment variables into the Render dashboard.
