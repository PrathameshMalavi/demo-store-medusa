# Medusa Architecture Overview

Medusa is built as a highly modular, headless commerce engine. Its architecture is separated into three primary components:

## High-Level Architecture

```mermaid
graph TD
    subgraph Frontend Storefronts
        Nextjs[Next.js Storefront]
        React[React Native App]
        OtherFrontend[Custom Frontend]
    end

    subgraph Medusa Backend (Node.js)
        API[REST APIs]
        AdminAPI[Admin APIs]
        StoreAPI[Storefront APIs]
        
        API --> AdminAPI
        API --> StoreAPI

        Core[Medusa Core Services]
        AdminAPI --> Core
        StoreAPI --> Core

        Modules[Modules & Plugins]
        Core --> Modules
        
        Modules --> AuthPlugin[Auth Module]
        Modules --> CartPlugin[Cart Module]
        Modules --> PaymentPlugin[Payment Providers e.g., Stripe]
        Modules --> SearchPlugin[Search Providers e.g., Meilisearch]
    end

    subgraph Database
        Postgres[(PostgreSQL)]
        Redis[(Redis - for events/queue)]
    end

    subgraph Admin Dashboard
        AdminUI[Medusa Admin UI]
    end

    Nextjs -->|REST| StoreAPI
    React -->|REST| StoreAPI
    OtherFrontend -->|REST| StoreAPI
    
    AdminUI -->|REST| AdminAPI

    Core --> Postgres
    Core --> Redis
```

## Key Components

### 1. Medusa Backend (Headless Engine)
This is the core Node.js application. It exposes two sets of REST APIs:
- **Admin API**: For managing the store (products, orders, customers, settings). Usually prefixed with `/admin`.
- **Store API**: For the customer-facing storefront (fetching products, managing carts, checkout). Usually prefixed with `/store`.

### 2. Admin Dashboard
A pre-built React application that consumes the Admin API. It provides a visual interface for store operators to manage products, view orders, and handle fulfillment.

### 3. PostgreSQL Database
The primary data store for all entities (users, products, orders, etc.). Medusa uses TypeORM internally to manage the database schema via migrations.

### 4. Redis (Optional but Recommended)
Used as an event bus and for job queuing (e.g., sending emails asynchronously, processing webhooks).

## Why Headless?

> [!NOTE]
> The headless architecture allows you to swap out any frontend without touching the backend logic. You can build a web storefront, an iOS app, and a POS system that all talk to the exact same Medusa backend APIs.
