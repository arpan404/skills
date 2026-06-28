---
name: author-js
description: Skill to build app authorization with author.js, including policies, backend enforcement, UI gates, stores, cache, management APIs, and tests.
version: 1.0.0
---

# author.js skill

Use this when adding authorization to an app.

author.js models:

- `entities`: actors, usually users or API clients
- `resources`: protected objects
- `actions`: operations like `read`, `update`, `delete`
- `policies`: named allow/deny rules
- `stores`: optional persistence for roles, permissions, relations, audit logs
- `cache`: optional decision cache, usually Redis

Backend checks are security. React checks only hide UI.

## Docs fallback

Use this skill first. If unsure, check:

- https://github.com/arpan404/author-js#readme
- https://github.com/arpan404/author-js/tree/main/docs
- https://www.npmjs.com/package/author-js

## Install

Use the app package manager:

```bash
bun add author-js
```

Add adapter peers only when used:

```bash
bun add pg mongodb redis react
```

Do not add DBs, Redis, or React unless the app already uses them or the user asks.

## Inspect first

Before coding, identify:

- framework and runtime/package manager
- auth/session source
- user shape: `id`, `role`, org/workspace/account ID
- protected resources and actions
- backend routes/actions needing enforcement
- UI controls to hide
- persistence needs: none, Postgres, MongoDB, Redis

Build the smallest layer that protects the requested feature.

## File location

Use app conventions. Good defaults:

```txt
src/authorization/author.ts
src/lib/author.ts
```

Start with one file.

## Core setup

```ts
import {
  allow,
  createAuthor,
  defineContext,
  defineEntity,
  defineResource,
} from "author-js";

type User = {
  id: string;
  role: "admin" | "member";
  organizationId?: string;
};

type Project = {
  id: string;
  ownerId: string;
  organizationId: string;
  status: "draft" | "published";
};

type AuthContext = {
  organizationId?: string;
  ip?: string;
};

const UserEntity = defineEntity<User>()({
  type: "User",
  id: (user) => user.id,
});

const ProjectResource = defineResource<Project>()({
  type: "Project",
  id: (project) => project.id,
  actions: ["read", "update", "delete"] as const,
});

const context = defineContext<AuthContext>();

export const author = createAuthor({
  context,
  entities: { User: UserEntity },
  resources: { Project: ProjectResource },
  policies: [
    allow("admins can do anything", ({ entity }) => entity.role === "admin"),

    allow("owners can update projects", ({ entity, resource, action }) => {
      if (resource.type !== "Project") return false;
      return action === "update" && resource.data.ownerId === entity.id;
    }),

    allow("members can read org projects", ({ entity, resource, action }) => {
      if (resource.type !== "Project") return false;
      return (
        action === "read" &&
        entity.organizationId === resource.data.organizationId
      );
    }),
  ],
});
```

## Permission checks

```ts
const allowed = await author
  .as("User", user)
  .can("update")
  .on("Project", project);

await author.as("User", user).can("update").on("Project", project).throw();

const decision = await author
  .as("User", user)
  .can("update")
  .on("Project", project)
  .explain();
```

Use explicit entity/resource names: `"User"`, `"Project"`.

Pass request context when needed:

```ts
await author
  .as("User", user)
  .can("read")
  .on("Project", project, { organizationId: "org_1" });
```

## Policy rules

Policies commonly receive:

```ts
({
  entity,
  resource,
  action,
  context,
  entityType,
  entityId,
  parents,
  subscription,
  features,
  limits,
});
```

Guidelines:

- Name policies after business rules.
- Return `false` when a policy does not apply.
- Check `resource.type` before reading `resource.data`.
- Use `deny` only when override behavior is required.
- Prefer small policies.
- Avoid `any`.

Common models:

```ts
// RBAC
allow("admins can delete projects", ({ entity, resource, action }) => {
  return (
    resource.type === "Project" &&
    action === "delete" &&
    entity.role === "admin"
  );
});

// ABAC
allow("published projects are readable", ({ resource, action }) => {
  return (
    resource.type === "Project" &&
    action === "read" &&
    resource.data.status === "published"
  );
});

// ReBAC
allow("project viewers can read projects", async (ctx) => {
  if (ctx.resource.type !== "Project" || ctx.action !== "read") return false;
  return ctx.parents.hasRelation({
    relation: "viewer",
    objectType: "Project",
    objectId: ctx.resource.id,
  });
});
```

## Stores and cache

No store is required for pure policy checks.

```ts
import { memoryStore } from "author-js";
import { postgresStore } from "author-js/postgres";
import { mongodbStore } from "author-js/mongodb";
import { redisCache } from "author-js/redis";
```

Use:

- `memoryStore()` for tests/prototypes
- `postgresStore({ connectionString: process.env.POSTGRES_URL! })` when the app uses Postgres
- `mongodbStore({ url: process.env.MONGODB_URL!, database: "app" })` when the app uses MongoDB
- `redisCache({ url: process.env.REDIS_URL!, prefix: "author:" })` only for real cache need

Example:

```ts
export const author = createAuthor({
  entities,
  resources,
  policies,
  store: postgresStore({ connectionString: process.env.POSTGRES_URL! }),
  cache: redisCache({ url: process.env.REDIS_URL!, prefix: "author:" }),
  cacheTtlMs: 30_000,
});
```

Use management helpers for writes so cache invalidation can happen.

## Management API

Use for admin screens, settings, seeds, and tests.

```ts
await author.roles.grant({ entityType, entityId, role, scopeType, scopeId });
await author.roles.list({ entityType, entityId, scopeType, scopeId });
await author.roles.revoke({ entityType, entityId, role, scopeType, scopeId });

await author.permissions.grant({
  entityType,
  entityId,
  action,
  resourceType,
  resourceId,
  effect: "allow",
});
await author.permissions.list({
  entityType,
  entityId,
  resourceType,
  resourceId,
});
await author.permissions.revoke({
  entityType,
  entityId,
  action,
  resourceType,
  resourceId,
});

await author.relations.create({
  subjectType,
  subjectId,
  relation,
  objectType,
  objectId,
});
await author.relations.list({ subjectType, subjectId, objectType, objectId });
await author.relations.delete({
  subjectType,
  subjectId,
  relation,
  objectType,
  objectId,
});
```

Prefer these helpers over raw DB writes.

## Parent access

Use when child resources inherit from org/workspace/folder/account.

```ts
allow("org admins can update projects", async (ctx) => {
  if (ctx.resource.type !== "Project" || ctx.action !== "update") return false;
  return ctx.parents.hasRole({
    role: "admin",
    objectType: "Organization",
    objectId: ctx.resource.data.organizationId,
  });
});
```

Helpers:

```ts
ctx.parents.getRequired({ objectType, objectId });
ctx.parents.hasRole({ role, objectType, objectId });
ctx.parents.hasPermission({ action, objectType, objectId });
ctx.parents.hasRelation({ relation, objectType, objectId });
```

## Entitlements

Use for plans, feature flags, and quotas.

```ts
allow("paid plans can export projects", async (ctx) => {
  if (ctx.resource.type !== "Project" || ctx.action !== "export") return false;
  return ctx.features.has("project_export");
});

allow("workspace is within project limit", async (ctx) => {
  return ctx.limits.within("projects", 1);
});
```

Helpers:

```ts
await ctx.subscription.plan();
await ctx.features.has("feature_name");
await ctx.features.list();
await ctx.limits.get("projects");
await ctx.limits.within("projects", 1);
await ctx.limits.remaining("projects");
```

## Backend enforcement

Always check before sensitive reads or mutations.

```ts
const user = await getCurrentUser(request);
const project = await getProject(params.projectId);

await author.as("User", user).can("update").on("Project", project).throw();

await updateProject(project.id, input);
```

Adapters:

```ts
import { requireCan } from "author-js/express";
import { requireCan } from "author-js/hono";
import { requireCan } from "author-js/fastify";
import { requireCan } from "author-js/elysia";
import { assertCan, requireCan } from "author-js/next/server";
```

Next.js server pattern:

```ts
await assertCan({
  author,
  entityType: "User",
  entity: user,
  action: "update",
  resourceType: "Project",
  resource: project,
});
```

## React UI

Use only for UI affordances.

```tsx
import { AuthorProvider, Can, Cannot, useCan } from "author-js/react";

<AuthorProvider authorization={author} entityType="User" entity={user}>
  {children}
</AuthorProvider>

<Can do="update" on="Project" resource={project} fallback={null}>
  <EditProjectButton />
</Can>

const { allowed, loading, decision } = useCan({
  do: "update",
  on: "Project",
  resource: project,
});
```

Next.js client import:

```tsx
import { AuthorProvider, Can, Cannot, useCan } from "author-js/next/client";
```

## Tests

Use the app test runner. Cover:

- one allowed case
- one denied case
- backend mutation rejection

Bun example:

```ts
import { expect, test } from "bun:test";
import { author } from "../src/authorization/author";

test("owners can update projects", async () => {
  const allowed = await author
    .as("User", { id: "u1", role: "member" })
    .can("update")
    .on("Project", {
      id: "p1",
      ownerId: "u1",
      organizationId: "org1",
      status: "draft",
    });

  expect(allowed).toBe(true);
});
```

## Workflow to work with author.js

1. Inspect stack, auth, models, resources, and routes.
2. Pick the smallest fitting model.
3. Create one `author` module.
4. Define only needed entities, resources, and actions.
5. Add named policies.
6. Enforce on backend before reads/mutations.
7. Add React gates only for UI.
8. Add store/cache only when needed.
9. Add focused tests.
10. Run relevant checks.

## Avoid

- React-only authorization
- replacing existing auth/session logic
- adding Redis without cache need
- adding unused resources/actions
- premature `PermissionService` wrappers
- raw DB writes instead of management helpers
- forgetting `deny` overrides `allow`
- optional context just to silence TypeScript
- reading `resource.data` before checking `resource.type`
