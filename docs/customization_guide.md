# Medusa Customization and Deployment Guide

## 1. Customizing Medusa

Medusa is highly extensible. If you want to add new fields to existing entities (like a `Product`) or create entirely new tables, you can do so by extending Medusa's models.

### Example: Adding a custom field to the Product model

> [!TIP]
> **What is a migration?** 
> Think of a migration as a "version control" script for your database. When you add `custom_image_url` to your code in Step 1, the database doesn't know about it yet. The migration script in Step 2 is what actually runs the `ALTER TABLE` SQL command to add that column to your PostgreSQL database.
> 
> **Does extending change the same DB table?** 
> Yes! When you extend the Medusa `Product` model, you are adding a new column directly to the existing `product` table in PostgreSQL.

1. **Create an extended entity:**
   In your root project folder (`src/models/product.ts`):
   ```typescript
   import { Column, Entity } from "typeorm"
   import { Product as MedusaProduct } from "@medusajs/medusa"

   @Entity()
   export class Product extends MedusaProduct {
     @Column()
     custom_image_url: string
   }
   ```

2. **Create a migration for the database:**
   To actually apply this change to the database, you create a migration script in `src/migrations/1688566085871-add_custom_image.ts`. When you run `npx medusa db:migrate`, Medusa will read this script and execute the SQL query inside `up()`:
   ```typescript
   import { MigrationInterface, QueryRunner } from "typeorm"

   export class addCustomImageUrlToProduct1688566085871 implements MigrationInterface {
       // The up() function contains the change you want to MAKE
       public async up(queryRunner: QueryRunner): Promise<void> {
           await queryRunner.query(`ALTER TABLE "product" ADD "custom_image_url" character varying`)
       }

       // The down() function contains the code to REVERT the change if you make a mistake
       public async down(queryRunner: QueryRunner): Promise<void> {
           await queryRunner.query(`ALTER TABLE "product" DROP COLUMN "custom_image_url"`)
       }
   }
   ```

3. **Update the service and API (Optional):**
   If you want to validate or manipulate this field when products are created via the API, you can override the `ProductService` in `src/services/product.ts` or create a subscriber that listens to the `product.created` event.

### Creating custom APIs
If you want to expose completely new API endpoints:
1. Create a file in `src/api/index.ts`.
2. Export a default function that takes an Express Router.
3. Define your routes:
   ```typescript
   import { Router } from "express"

   export default (rootDirectory: string): Router | Router[] => {
     const router = Router()
     
     router.get("/custom", (req, res) => {
       res.json({
         message: "This is a custom endpoint",
       })
     })

     return router
   }
   ```

## 2. Switching to Supabase for Deployment

Supabase provides a powerful PostgreSQL database that works perfectly with Medusa.

### Steps to migrate from Local Postgres to Supabase

1. **Create a Supabase Project:**
   - Go to [Supabase](https://supabase.com) and create a new project.
   - Wait for the database to provision.

2. **Get the Database Connection String:**
   - In your Supabase dashboard, navigate to **Project Settings -> Database**.
   - Copy the **Connection String (URI)**.
   - It will look something like this: `postgres://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres`

> [!WARNING]
> Supabase uses a connection pooler by default (port 6543). For running migrations, it's highly recommended to use the direct connection string (port 5432) to avoid transaction pooling issues. The direct string looks like:
> `postgres://postgres:[password]@db.[project-ref].supabase.co:5432/postgres`

3. **Update your `.env` file:**
   - In your deployed environment (or locally if testing), update the `.env` file for your Medusa backend:
   ```env
   # Replace your local URL with the Supabase direct connection string
   DATABASE_URL="postgres://postgres:[password]@db.[project-ref].supabase.co:5432/postgres"
   ```

4. **Run Migrations on Supabase:**
   - Run the Medusa migrations against the Supabase database to create all the necessary tables:
   ```bash
   npx medusa migrations run
   ```

5. **Deploy:**
   - Once your tables are set up in Supabase, you can deploy your Medusa backend (Node.js app) to platforms like Railway, Render, or Heroku.
   - Ensure the `DATABASE_URL` environment variable on your hosting platform is set to the Supabase connection string.
