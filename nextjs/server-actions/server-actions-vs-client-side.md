- [Should all database calls be server actions?](#should-all-database-calls-be-server-actions)
  - [**Why Use Server Actions for Database Calls?**](#why-use-server-actions-for-database-calls)
  - [**When to Avoid Server Actions?**](#when-to-avoid-server-actions)
  - [**Recommended Approach**](#recommended-approach)
    - [**1. For Sensitive or Authenticated Data:**](#1-for-sensitive-or-authenticated-data)
    - [**2. For Realtime Updates:**](#2-for-realtime-updates)
    - [**3. For Static or Public Data:**](#3-for-static-or-public-data)
  - [**Decision Framework**](#decision-framework)
- [How to Avoid Duplicate Code in Server Actions and Client-side Code](#how-to-avoid-duplicate-code-in-server-actions-and-client-side-code)
  - [**1. Extract Shared Logic into Utility Functions**](#1-extract-shared-logic-into-utility-functions)
    - [Example:](#example)
    - [Server Action:](#server-action)
    - [Client Code:](#client-code)
  - [**2. Use Dependency Injection for Supabase Client**](#2-use-dependency-injection-for-supabase-client)
    - [Example:](#example-1)
    - [Server Action:](#server-action-1)
    - [Client Code:](#client-code-1)
  - [**3. Centralize Data Fetching via an API Route**](#3-centralize-data-fetching-via-an-api-route)
    - [Example:](#example-2)
    - [Client Code (Fetch Data from API):](#client-code-fetch-data-from-api)
  - [**4. Combine with Supabase Functions**](#4-combine-with-supabase-functions)
  - [**Summary of Approaches**](#summary-of-approaches)
- [Service vs Repository vs Controller vs Manager](#service-vs-repository-vs-controller-vs-manager)
  - [**1. Service**](#1-service)
  - [**2. Repository**](#2-repository)
  - [**3. Controller**](#3-controller)
  - [**4. Manager**](#4-manager)
  - [**Which to Choose?**](#which-to-choose)
  - [**Recommended Approach for Supabase in Next.js**](#recommended-approach-for-supabase-in-nextjs)

## Should all database calls be server actions?

In Next.js (with TypeScript and Supabase), whether all database calls should be implemented as server actions depends on your use case and architectural considerations. Here's a breakdown of best practices:

---

### **Why Use Server Actions for Database Calls?**

1. **Security**:

   - Keeping database calls on the server prevents exposing your Supabase API key or other sensitive credentials to the client.
   - This minimizes risks of unauthorized access or misuse.

2. **Reduced Client-Side Logic**:

   - Server-side logic centralizes data-fetching and processing, keeping the client-side code cleaner and focused on presentation.

3. **Better Performance**:

   - Server actions leverage server-side rendering (SSR) or static site generation (SSG) for efficient, optimized responses.

4. **Integration with Middleware**:
   - Server actions make it easier to integrate additional layers, such as authentication checks, logging, or caching.

---

### **When to Avoid Server Actions?**

1. **Realtime Applications**:

   - For features like chat, notifications, or live updates, consider using Supabase's **Realtime** capabilities directly on the client to reduce latency and enable live data synchronization.

2. **Lightweight Client Queries**:

   - For simple public queries (e.g., fetching a list of posts without authentication), it may be acceptable to call Supabase directly from the client.

3. **Reduced Server Load**:
   - Moving less-critical queries to the client reduces the load on your server and can improve scalability for highly-trafficked apps.

---

### **Recommended Approach**

#### **1. For Sensitive or Authenticated Data:**

- Use **Server Actions** or API routes in Next.js (`app/api`).
- Example:

  ```typescript
  import { createServerAction } from "next/server";
  import { createClient } from "@supabase/supabase-js";

  export const fetchUserData = createServerAction(async (userId: string) => {
    const supabase = createClient(SUPABASE_URL, SUPABASE_SECRET_KEY);
    const { data, error } = await supabase
      .from("users")
      .select("*")
      .eq("id", userId);
    if (error) throw new Error(error.message);
    return data;
  });
  ```

#### **2. For Realtime Updates:**

- Leverage **Supabase Client-Side Realtime API**.
- Example:

  ```typescript
  import { createClient } from "@supabase/supabase-js";

  const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

  const channel = supabase
    .channel("public:posts")
    .on(
      "postgres_changes",
      { event: "*", schema: "public", table: "posts" },
      (payload) => {
        console.log("Change received!", payload);
      }
    )
    .subscribe();
  ```

#### **3. For Static or Public Data:**

- Use **Static Generation** (SSG) or **Incremental Static Regeneration** (ISR) for static pages.
- Example:

  ```typescript
  import { createClient } from "@supabase/supabase-js";

  export const getStaticProps = async () => {
    const supabase = createClient(SUPABASE_URL, SUPABASE_SECRET_KEY);
    const { data, error } = await supabase.from("posts").select("*");
    if (error) {
      return { notFound: true };
    }
    return { props: { posts: data }, revalidate: 60 }; // Regenerate every 60 seconds
  };
  ```

---

### **Decision Framework**

| Use Case                  | Approach                          |
| ------------------------- | --------------------------------- |
| Secure/Authenticated Data | Server actions or API routes      |
| Public/Static Data        | SSG or ISR                        |
| Realtime Functionality    | Supabase Realtime on the client   |
| Heavy Processing Logic    | Offload to server actions or APIs |

By default, prioritize server actions for database interactions unless client-side real-time features or public data needs dictate otherwise.

## How to Avoid Duplicate Code in Server Actions and Client-side Code

To avoid duplicated code when you need both server actions and client-side code for similar database interactions, you can refactor the shared logic into reusable functions or modules. Here’s how you can organize your code effectively:

---

### **1. Extract Shared Logic into Utility Functions**

Create a utility function for the database operation that can be reused in both the server and client contexts.

#### Example:

```typescript
// utils/database.ts
import { createClient } from "@supabase/supabase-js";

const supabaseClient = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_KEY!
);

export const fetchUserById = async (userId: string) => {
  const { data, error } = await supabaseClient
    .from("users")
    .select("*")
    .eq("id", userId)
    .single();
  if (error) throw new Error(error.message);
  return data;
};
```

Now, use this function in both server and client code:

#### Server Action:

```typescript
// app/actions/fetchUser.ts
import { fetchUserById } from "@/utils/database";

export const fetchUser = async (userId: string) => {
  return await fetchUserById(userId);
};
```

#### Client Code:

```typescript
// app/page.tsx
"use client";

import { fetchUserById } from "@/utils/database";

const UserProfile = async ({ userId }: { userId: string }) => {
  try {
    const user = await fetchUserById(userId);
    return <div>{user.name}</div>;
  } catch (error) {
    return <div>Error: {error.message}</div>;
  }
};
```

---

### **2. Use Dependency Injection for Supabase Client**

Pass the Supabase client as an argument to the utility function. This allows you to use the same logic in different contexts (server or client).

#### Example:

```typescript
// utils/database.ts
export const fetchUser = async (supabase: SupabaseClient, userId: string) => {
  const { data, error } = await supabase
    .from("users")
    .select("*")
    .eq("id", userId)
    .single();
  if (error) throw new Error(error.message);
  return data;
};
```

#### Server Action:

```typescript
import { createClient } from "@supabase/supabase-js";
import { fetchUser } from "@/utils/database";

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_KEY!
);

export const fetchUserServer = async (userId: string) => {
  return await fetchUser(supabase, userId);
};
```

#### Client Code:

```typescript
import { createClient } from "@supabase/supabase-js";
import { fetchUser } from "@/utils/database";

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
);

const fetchUserClient = async (userId: string) => {
  return await fetchUser(supabase, userId);
};
```

---

### **3. Centralize Data Fetching via an API Route**

Instead of writing separate client and server-side logic, you can centralize the database calls in an API route. This approach avoids duplication altogether.

#### Example:

```typescript
// app/api/user/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { fetchUserById } from "@/utils/database";

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await fetchUserById(params.id);
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.error();
  }
}
```

#### Client Code (Fetch Data from API):

```typescript
const fetchUser = async (userId: string) => {
  const response = await fetch(`/api/user/${userId}`);
  if (!response.ok) throw new Error("Failed to fetch user");
  return await response.json();
};
```

---

### **4. Combine with Supabase Functions**

For even more modularity, consider moving logic to **Supabase Edge Functions**. These functions can encapsulate the database logic and be called from both the server and client securely.

---

### **Summary of Approaches**

| Approach                         | When to Use                                                             |
| -------------------------------- | ----------------------------------------------------------------------- |
| Utility functions (shared logic) | Simple use cases where client and server need the same operation.       |
| Dependency injection             | To support context-specific Supabase clients (e.g., private vs public). |
| Centralized API routes           | When avoiding client-side logic duplication entirely.                   |
| Supabase Edge Functions          | For highly reusable, secure, serverless logic.                          |

By centralizing the database interaction in a reusable function or API route, you maintain consistency while avoiding code duplication.

## Service vs Repository vs Controller vs Manager

When centralizing your database logic in a class, the naming convention you choose should align with its role and responsibilities. Here's a breakdown of common naming patterns:

---

### **1. Service**

- **Use Case**: A service class encapsulates business logic and interactions with the database.
- **Common in**: Backend-focused architecture (e.g., Node.js, NestJS, Spring Boot).
- **When to Use**: If your class is primarily responsible for implementing reusable and high-level operations that interact with the database.
- **Example**:

  ```typescript
  // services/UserService.ts
  import { SupabaseClient } from "@supabase/supabase-js";

  export class UserService {
    private supabase: SupabaseClient;

    constructor(supabase: SupabaseClient) {
      this.supabase = supabase;
    }

    async getUserById(userId: string) {
      const { data, error } = await this.supabase
        .from("users")
        .select("*")
        .eq("id", userId)
        .single();
      if (error) throw new Error(error.message);
      return data;
    }
  }
  ```

---

### **2. Repository**

- **Use Case**: A repository class focuses purely on data access, abstracting database queries.
- **Common in**: Data Access Layer (DAL) implementations, particularly in systems with layered architecture.
- **When to Use**: If your class is limited to database interactions without any business logic.
- **Example**:

  ```typescript
  // repositories/UserRepository.ts
  import { SupabaseClient } from "@supabase/supabase-js";

  export class UserRepository {
    private supabase: SupabaseClient;

    constructor(supabase: SupabaseClient) {
      this.supabase = supabase;
    }

    async fetchUserById(userId: string) {
      const { data, error } = await this.supabase
        .from("users")
        .select("*")
        .eq("id", userId)
        .single();
      if (error) throw new Error(error.message);
      return data;
    }
  }
  ```

---

### **3. Controller**

- **Use Case**: A controller is typically responsible for handling requests, orchestrating services, and returning responses.
- **Common in**: MVC (Model-View-Controller) frameworks.
- **When to Use**: If your class is tied to handling client requests and delegating logic to services or repositories. In the context of Next.js, this would likely be unnecessary because API routes or server actions act as controllers.
- **Example**:

  ```typescript
  // controllers/UserController.ts
  import { UserService } from "@/services/UserService";

  export class UserController {
    private userService: UserService;

    constructor(userService: UserService) {
      this.userService = userService;
    }

    async getUser(userId: string) {
      return await this.userService.getUserById(userId);
    }
  }
  ```

---

### **4. Manager**

- **Use Case**: A manager class combines logic from multiple services, repositories, or external APIs.
- **Common in**: Complex systems where you need to orchestrate multiple dependencies.
- **When to Use**: If your class combines business logic from multiple sources.
- **Example**:

  ```typescript
  // managers/UserManager.ts
  import { UserService } from "@/services/UserService";
  import { AnalyticsService } from "@/services/AnalyticsService";

  export class UserManager {
    private userService: UserService;
    private analyticsService: AnalyticsService;

    constructor(userService: UserService, analyticsService: AnalyticsService) {
      this.userService = userService;
      this.analyticsService = analyticsService;
    }

    async getUserWithStats(userId: string) {
      const user = await this.userService.getUserById(userId);
      const stats = await this.analyticsService.getUserStats(userId);
      return { ...user, stats };
    }
  }
  ```

---

### **Which to Choose?**

- **Service**: When focusing on high-level operations with some business logic.
- **Repository**: When isolating pure database access without additional logic.
- **Controller**: Rarely used for centralizing database logic in Next.js—API routes or server actions act as controllers.
- **Manager**: When coordinating multiple services or external systems.

---

### **Recommended Approach for Supabase in Next.js**

- **Name it `UserService` or similar**, as it encapsulates business logic and database operations in a meaningful, reusable way. This is a common and intuitive choice for this use case.

```typescript
// services/UserService.ts
import { SupabaseClient } from "@supabase/supabase-js";

export class UserService {
  private supabase: SupabaseClient;

  constructor(supabase: SupabaseClient) {
    this.supabase = supabase;
  }

  async getUserById(userId: string) {
    const { data, error } = await this.supabase
      .from("users")
      .select("*")
      .eq("id", userId)
      .single();
    if (error) throw new Error(error.message);
    return data;
  }
}
```

This provides clarity to team members that the class encapsulates reusable logic to work with user data.
