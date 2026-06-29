---
name: author-js
description: Skill to build app authorization with author.js, including scoped policies, modules, backend enforcement, UI gates, stores, cache, management APIs, audit hooks, and tests.
version: 2.0.0
---

# author.js skill

Use this when adding authorization to an app.

author.js models:

- `entities`: actors, usually users or API clients
- `resources`: protected objects with typed actions and optional parents
- `modules`: domain groups of resources and scoped policies
- `policies`: named allow/deny rules selected by entity type, resource type, and action
- `stores`: optional persistence for roles, permissions, relations, and audit logs
- `cache`: optional short-lived decision cache
- `entitlements`: optional plans, features, and limits
- `hooks`: optional `afterDecision` side effects for metrics, logs, tracing, or cache warming

Backend checks are security. React checks only hide UI.

## Docs fallback

Use this skill first. If unsure, check the current upstream docs:

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
- user shape: `id`, `role`, org/workspace/account ID, plan if relevant
- protected resources and actions
- parent relationships such as organization, workspace, folder, or account
- backend routes/actions needing enforcement
- UI controls to hide
- persistence needs: none, memory, Postgres, MongoDB, Redis
- audit needs: `all`, `explain`, or `none`

Build the smallest layer that protects the requested feature.

## File location

Use app conventions. Good defaults:

```txt
src/authorization/author.ts
src/lib/author.ts
```

For larger apps:

```txt
src/authorization/
  author.ts
  definitions.ts
  plans.ts
  modules/
    projects.ts
    billing.ts
  policies/
    organization.ts
```

Start with one file unless the app already has authorization structure or multiple domains.

## Core setup

Prefer `defineAuthorModule` and the scoped `policy` builder. Scopes are static applicability metadata; dynamic checks stay inside the policy function.

```ts
import {
  createAuthor,
  defineAuthorModule,
  defineContext,
  defineEntity,
  defineResource,
  policy,
} from "author-js";

type User = {
  id: string;
  role: "admin" | "member";
  organizationId?: string;
  plan: "free" | "pro" | "enterprise";
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
  projectCount?: number;
};

const UserEntity = defineEntity<User>()({
  type: "User",
  id: (user) => user.id,
});

const ProjectResource = defineResource<Project>()({
  type: "Project",
  id: (project) => project.id,
  actions: ["read", "create", "update", "delete", "export"] as const,
  parents: {
    organization: {
      type: "Organization",
      id: (project) => project.organizationId,
    },
  },
});

const AuthContext = defineContext<AuthContext>();

const projectModule = defineAuthorModule({
  name: "projects",
  resources: { Project: ProjectResource },
  policies: [
    policy
      .for("User")
      .on("Project")
      .can(["read", "create", "update", "delete", "export"])
      .allow("admins can do anything", ({ entity }) => entity.role === "admin"),

    policy
      .for("User")
      .on("Project")
      .can("update")
      .allow("owners can update projects", ({ entity, resource }) => {
        return resource.data.ownerId === entity.id;
      }),

    policy
      .for("User")
      .on("Project")
      .can("read")
      .allow("members can read org projects", ({ entity, resource }) => {
        return entity.organizationId === resource.data.organizationId;
      }),

    policy
      .for("User")
      .on("Project")
      .can("delete")
      .deny("members cannot delete projects", ({ entity }) => {
        return entity.role === "member";
      }),
  ],
});

export const author = createAuthor({
  context: AuthContext,
  entities: { User: UserEntity },
  modules: [projectModule],
});
```

For tiny apps, `createAuthor({ resources, policies })` is still valid. Prefer modules when resources and policies are owned by domains.

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
await author.as("User", user).can("create").on("Project", project, {
  organizationId: "org_1",
  projectCount: 4,
});
```

Use direct checks when a framework helper or service function is easier to compose:

```ts
const allowed = await author.check({
  entityType: "User",
  entity: user,
  action: "update",
  resourceType: "Project",
  resource: project,
  context: {},
  mode: "backend",
});
```

`.throw()` raises `AuthorizationDeniedError` when denied.

## Policy rules

Policies commonly receive:

```ts
({
  entity,
  resource,
  action,
  context,
  mode,
  entityType,
  entityId,
  roles,
  permissions,
  relations,
  parents,
  subscription,
  features,
  limits,
  entityHasRelation,
});
```

Guidelines:

- Name policies after business rules.
- Use `policy.for(...).on(...).can(...).allow(...)` or `.deny(...)`.
- Scope policies by entity type, resource type, and action in non-trivial apps.
- Return `false` when a policy does not apply.
- Use `deny` for explicit override behavior; boolean checks evaluate relevant denies before allows.
- Keep dynamic checks such as ownership, tenant membership, roles, and subscription state inside the policy function.
- Prefer small policies.
- Avoid `any`.
- Use `.explain()` when you need all matching and skipped policies.

Common models:

```ts
// RBAC
policy
  .for("User")
  .on("Project")
  .can("delete")
  .allow("admins can delete projects", ({ entity }) => {
    return entity.role === "admin";
  });

// ABAC
policy
  .for("User")
  .on("Project")
  .can("read")
  .allow("published projects are readable", ({ resource }) => {
    return resource.data.status === "published";
  });

// ReBAC
policy
  .for("User")
  .on("Project")
  .can("read")
  .allow("project viewers can read projects", async (ctx) => {
    return ctx.relations.has({
      subjectType: "User",
      subjectId: ctx.entityId,
      relation: "viewer",
      objectType: "Project",
      objectId: ctx.resource.id,
    });
  });
```

Lower-level helpers are still available for generated scopes:

```ts
import { allow, deny } from "author-js";

allow("name", { entityTypes: ["User"], resourceTypes: ["Project"], actions: ["read"] }, check);
deny("name", { entityTypes: ["User"], resourceTypes: ["Project"], actions: ["delete"] }, check);
```

## Decision hooks and audit

Use `afterDecision` for side effects. Hooks cannot change the decision.

```ts
policy
  .for("User")
  .on("Project")
  .can("read")
  .afterDecision("record project read", async (ctx, decision) => {
    await metrics.count("auth.project.read", {
      allowed: decision.allowed,
      mode: ctx.mode,
    });
  });
```

For global hooks, use `afterDecision(name, run)` directly or omit builder scopes.

Stores with `writeAuditLog` receive audit entries by default. Tune per author instance:

```ts
export const author = createAuthor({
  entities,
  modules,
  audit: "explain",
});
```

Audit modes:

- `all`: log boolean checks and explanations
- `explain`: log only `.explain()` and `author.evaluate(...)`
- `none`: disable automatic audit writes

## Stores and cache

No store is required for pure policy checks.

```ts
import { memoryCache, memoryStore } from "author-js";
import { postgresStore } from "author-js/postgres";
import { ensureMongoIndexes, mongodbStore } from "author-js/mongodb";
import { redisCache } from "author-js/redis";
```

Use:

- `memoryStore()` for tests/prototypes
- `memoryCache()` for process-local decision caching
- `postgresStore({ connectionString: process.env.DATABASE_URL! })` when the app uses Postgres
- `postgresStore({ client })` to reuse a pg-compatible client
- `mongodbStore({ client, database: "app" })` when the app uses MongoDB
- `ensureMongoIndexes({ client, database: "app" })` during setup/migrations
- `redisCache({ client, prefix: "author:" })` only for real shared cache need

Example:

```ts
export const author = createAuthor({
  entities,
  modules,
  store: postgresStore({ connectionString: process.env.DATABASE_URL! }),
  cache: redisCache({ client: redis, prefix: "author:" }),
  cacheTtlMs: 30_000,
  audit: "explain",
});
```

Use management helpers for writes so cache invalidation can happen.

Advanced cache options:

```ts
export const author = createAuthor({
  entities,
  modules,
  cache,
  cacheTtlMs: 30_000,
  cacheKey: ({ entityType, entityId, action, resourceType, resourceId }) => {
    return `${entityType}:${entityId}:${action}:${resourceType}:${resourceId}`;
  },
});

await author.invalidate();
```

## Management API

Use for admin screens, settings, seeds, billing webhooks, and tests. Prefer these helpers over raw DB writes.

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
  effect: "allow",
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

Permission policies decide how direct permissions are used:

```ts
policy
  .for("User")
  .on("Project")
  .can("read")
  .allow("direct permission can read project", async (ctx) => {
    return ctx.permissions.has("read", {
      type: "Project",
      id: ctx.resource.id,
    });
  });
```

`ctx.permissions.has` returns `false` when a matching deny exists.

## Parent access

Use when child resources inherit from org/workspace/folder/account. Parents must be declared on the resource.

```ts
const ProjectResource = defineResource<Project>()({
  type: "Project",
  id: (project) => project.id,
  actions: ["read", "update"] as const,
  parents: {
    organization: {
      type: "Organization",
      id: (project) => project.organizationId,
    },
  },
});

policy
  .for("User")
  .on("Project")
  .can("update")
  .allow("org admins can update projects", async (ctx) => {
    return ctx.parents.hasRole("admin", "organization");
  });
```

Helpers:

```ts
await ctx.parents.get("organization");
await ctx.parents.getRequired("organization");
await ctx.parents.list();
await ctx.parents.hasRole("admin", "organization");
await ctx.parents.hasPermission("read", "organization");
await ctx.parents.hasRelation("member", "organization");
```

Parent permissions are not inherited automatically. Write explicit rules.

## Entitlements

Use for plans, feature flags, and quotas.

```ts
const plans = {
  free: {
    features: ["projects.read"],
    limits: { projects: 3 },
  },
  pro: {
    features: ["projects.read", "projects.create", "projects.export"],
    limits: { projects: 100 },
  },
} as const;

export const entitlements = {
  plan: ({ entity }: { entity: { plan: keyof typeof plans } }) => entity.plan,
  features: Object.fromEntries(
    Object.entries(plans).map(([name, plan]) => [name, plan.features]),
  ),
  limits: Object.fromEntries(
    Object.entries(plans).map(([name, plan]) => [name, plan.limits]),
  ),
};

export const author = createAuthor({
  entities,
  modules,
  entitlements,
});
```

Use entitlement helpers in policies:

```ts
policy
  .for("User")
  .on("Project")
  .can("create")
  .allow("plan can create projects", async (ctx) => {
    if (!(await ctx.features.has("projects.create"))) return false;

    const used = Number(ctx.context.projectCount ?? 0);
    return ctx.limits.within("projects", { used });
  });
```

Helpers:

```ts
await ctx.subscription.plan();
await ctx.features.has("projects.create");
await ctx.features.list();
await ctx.limits.get("projects");
await ctx.limits.within("projects", { used: 2 });
await ctx.limits.remaining("projects", { used: 2 });
```

Pass usage through context or query it inside the policy.

## Backend enforcement

Always check before sensitive reads or mutations. Authorize against the loaded resource, not route params alone.

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

Express/Hono/Fastify/Elysia adapters:

- resolve the entity from the request
- load the resource from the database
- evaluate in `backend` mode
- return 403 when denied

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

Reusable Next.js check:

```ts
const canUpdateProject = requireCan({
  author,
  entityType: "User",
  entity: async () => getCurrentUser(),
  action: "update",
  resourceType: "Project",
  resource: async (request) => loadProject(getProjectId(request)),
});

await canUpdateProject(request);
```

## React UI

Use only for UI affordances.

```tsx
import {
  AuthorProvider,
  Can,
  Cannot,
  useAuthor,
  useCan,
  useCannot,
} from "author-js/react";

<AuthorProvider
  authorization={author}
  entityType="User"
  entity={user}
  context={{ tenantId }}
>
  {children}
</AuthorProvider>

<Can do="update" on="Project" resource={project} fallback={null}>
  <EditProjectButton />
</Can>

<Cannot do="delete" on="Project" resource={project}>
  <DeleteUnavailable />
</Cannot>

const { allowed, loading, error, decision } = useCan({
  do: "update",
  on: "Project",
  resource: project,
});

const { allowed: isDenied } = useCannot({
  do: "delete",
  on: "Project",
  resource: project,
});

const { authorization, entity, mode, context } = useAuthor();
```

Use `i` to check a different actor than the provider default:

```tsx
<Can i={serviceAccount} do="read" on="Project" resource={project}>
  <ServiceAccountBadge />
</Can>
```

Component context merges with provider context. Component keys override provider keys.

Next.js client import:

```tsx
import { AuthorProvider, Can, Cannot, useCan } from "author-js/next/client";
```

## Tests

Use the app test runner. Cover:

- one allowed case
- one denied case
- backend mutation rejection
- management helper behavior if roles, permissions, or relations are used
- hook/audit behavior if side effects matter

Bun example:

```ts
import { expect, test } from "bun:test";
import { author } from "../src/authorization/author";

test("owners can update projects", async () => {
  const allowed = await author
    .as("User", { id: "u1", role: "member", plan: "pro" })
    .can("update")
    .on("Project", {
      id: "p1",
      ownerId: "u1",
      organizationId: "org1",
      status: "draft",
    });

  expect(allowed).toBe(true);
});

test("members cannot delete projects", async () => {
  const decision = await author
    .as("User", { id: "u1", role: "member", plan: "pro" })
    .can("delete")
    .on("Project", {
      id: "p1",
      ownerId: "u1",
      organizationId: "org1",
      status: "draft",
    })
    .explain();

  expect(decision.allowed).toBe(false);
});
```

For author.js itself, upstream uses:

```bash
bun run check
bun run fmt:check
bun run lint
bun run typecheck
docker compose up -d --wait
bun run test:integration
docker compose down
```

Run the relevant checks for the app you are editing.

## Workflow to work with author.js

1. Inspect stack, auth, models, resources, parent relationships, and routes.
2. Pick the smallest fitting model.
3. Create one `author` module.
4. Define only needed entities, resources, parents, and actions.
5. Add named scoped policies with the `policy` builder.
6. Compose modules once in `createAuthor`.
7. Enforce on backend before reads/mutations.
8. Add React gates only for UI.
9. Add store/cache/audit only when needed.
10. Add focused tests.
11. Run relevant checks.

## Avoid

- React-only authorization
- replacing existing auth/session logic
- adding Redis without cache need
- adding unused resources/actions
- premature `PermissionService` wrappers
- raw DB writes instead of management helpers
- forgetting relevant `deny` rules are checked before allows
- optional context just to silence TypeScript
- relying on route params instead of loaded resources
- expecting parent permissions to inherit without explicit policy rules
