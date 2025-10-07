# Routing Guide

This guide covers both **OpenAPI routes** (documented in Swagger) and **Normal Hono routes** (not documented).

## Table of Contents

- [Structure](#structure)
- [Route Types](#route-types)
  - [1. OpenAPI Routes (Documented in Swagger)](#1-openapi-routes-documented-in-swagger)
  - [2. Normal Hono Routes (Not Documented)](#2-normal-hono-routes-not-documented)
- [Adding New Resource (OpenAPI)](#adding-new-resource-openapi)
  - [1. Create Schema File](#1-create-schema-file)
  - [2. Export Schema](#2-export-schema)
  - [3. Setup Routes](#3-setup-routes)
  - [4. Register Routes](#4-register-routes)
- [Adding New Resource (Normal Hono Routes)](#adding-new-resource-normal-hono-routes)
  - [1. Create Routes File](#1-create-routes-file)
  - [2. Register Routes](#2-register-routes)
- [Available Helpers](#available-helpers)
- [Tags Usage](#tags-usage)
- [Type Inference Pattern](#type-inference-pattern)

## Structure

```tree
src/
├── schemas/                    # Zod schemas + OpenAPI route definitions
├── routes/                     # Route setup (both types)
│   └── index.ts                # Can mount both types
├── modules/
│   └── _example/               # Example module
│       └── routes/
│           └── product.routes.ts
└── utils/openapi-helpers.ts    # OpenAPI helper functions
```

## Route Types

### 1. OpenAPI Routes (Documented in Swagger)

- Defined using `app.openapi()` with Zod schemas
- Automatically appear in `/swagger` UI
- Type-safe with request/response validation

### 2. Normal Hono Routes (Not Documented)

- Defined using standard Hono methods (`app.get()`, `app.post()`, etc.)
- Still fully functional but won't appear in Swagger

---

## Adding New Resource (OpenAPI)

### 1. Create Schema File

`/schemas/post.schemas.ts`:

```typescript
import { z } from "zod";
import {
  createGetRoute,
  createPostRoute,
  createPutRoute,
  createDeleteRoute,
} from "@/utils/openapi-helpers";

const PostSchema = z.object({
  id: z.uuid(),
  title: z.string(),
  content: z.string(),
});

const CreatePostSchema = z.object({
  title: z.string(),
  content: z.string(),
});

const PostIdParam = z.object({
  id: z.uuid(),
});

const getPostsRoute = createGetRoute({
  path: "/posts",
  summary: "Get posts",
  responseSchema: z.array(PostSchema),
  tags: ["Posts"],
});

const getPostRoute = createGetRoute({
  path: "/posts/{id}",
  summary: "Get post",
  responseSchema: PostSchema,
  params: PostIdParam,
  tags: ["Posts"],
});

const createPostRoute = createPostRoute({
  path: "/posts",
  summary: "Create post",
  requestSchema: CreatePostSchema,
  responseSchema: PostSchema,
  tags: ["Posts"],
});

const updatePostRoute = createPutRoute({
  path: "/posts/{id}",
  summary: "Update post",
  requestSchema: CreatePostSchema.partial(),
  responseSchema: PostSchema,
  params: PostIdParam,
  tags: ["Posts"],
});

const deletePostRoute = createDeleteRoute({
  path: "/posts/{id}",
  summary: "Delete post",
  params: PostIdParam,
  tags: ["Posts"],
});

export {
  PostSchema,
  CreatePostSchema,
  PostIdParam,
  getPostsRoute,
  getPostRoute,
  createPostRoute,
  updatePostRoute,
  deletePostRoute,
};
```

### 2. Export Schema

Add to `/schemas/index.ts`:

```typescript
export * as PostSchemas from "./post.schemas";
```

### 3. Setup Routes

`/routes/post.routes.ts`

```typescript
import type { OpenAPIHono } from "@hono/zod-openapi";
import { PostSchemas } from "@/schemas";
import { PostController } from "@/controllers";

const setupPostRoutes = (app: OpenAPIHono) => {
  app.openapi(PostSchemas.getPostsRoute, PostController.getPosts);
  app.openapi(PostSchemas.getPostRoute, PostController.getPost);
  app.openapi(PostSchemas.createPostRoute, PostController.createPost);
  app.openapi(PostSchemas.updatePostRoute, PostController.updatePost);
  app.openapi(PostSchemas.deletePostRoute, PostController.deletePost);
};

export { setupPostRoutes };
```

### 4. Register Routes

Add to `/routes/index.ts`:

```typescript
import { setupPostRoutes } from "./post.openapi.routes";

export const setupRoutes = (app: OpenAPIHono) => {
  // ============================================
  // OpenAPI Routes (documented in Swagger)
  // ============================================
  setupPostRoutes(app);
};
```

---

## Adding New Resource (Normal Hono Routes)

### 1. Create Routes File

`/routes/product.routes.ts`:

```typescript
import { Hono } from "hono";
import { ProductController } from "@/controllers";

const productRoutes = new Hono();

productRoutes.get("/:id", ProductController.getProduct);
productRoutes.post("/", ProductController.createProduct);

export { productRoutes };
```

### 2. Register Routes

Add to `/routes/index.ts`:

```typescript
import { productRoutes } from "@/modules/_example";

export const setupRoutes = (app: OpenAPIHono) => {
  // OpenAPI Routes (documented in Swagger)
  // setupPostRoutes(app);

  // Normal Hono Routes (not in Swagger docs)
  app.route("/products", productRoutes);
};
```

## Available Helpers

- `createGetRoute` - GET endpoints
- `createPostRoute` - POST endpoints
- `createPutRoute` - PUT endpoints
- `createDeleteRoute` - DELETE endpoints

## Tags Usage

All helper functions support an optional `tags` parameter for OpenAPI documentation organization.

```typescript
const getPostsRoute = createGetRoute({
  path: "/posts",
  summary: "Get all posts",
  responseSchema: z.array(PostSchema),
  tags: ["Posts"], // Groups endpoints in OpenAPI docs
});

const createPostRoute = createPostRoute({
  path: "/posts",
  summary: "Create post",
  requestSchema: CreatePostSchema,
  responseSchema: PostSchema,
  tags: ["Posts", "Content"], // Multiple tags supported
});
```

Tags help organize your API documentation by grouping related endpoints together in the OpenAPI/Swagger UI.

## Type Inference Pattern

Just simply infer types from Schemas using Zod

`/types/post.types.ts`:

```typescript
import { z } from "zod";
import {
  PostSchema,
  CreatePostSchema,
  PostIdParam,
} from "@/schemas/post.schemas";

export type Post = z.infer<typeof PostSchema>;
export type CreatePost = z.infer<typeof CreatePostSchema>;
export type PostId = z.infer<typeof PostIdParam>;
```

```typescript
export type ResourceName = z.infer<typeof ResourceSchema>;
export type CreateResourceName = z.infer<typeof CreateResourceSchema>;
```
