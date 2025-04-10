# Renton Technical College CSI-246

<div align="center">  
    <img src="logo.jpg" alt="Logo">
    <h3 align="center">Guided Activity 2</h3>
</div>

This repository is a part of CSI-246 at Renton Technical College.

## Getting Started

1. Clone the repository to your local machine:

2. Open the cloned repository in VS Code:

3. Follow the instructions below to complete the assignment. We'll be building a blog application using Next.js 15's new features including the App Router, Server Components, and more.

## Part 1: Understanding Next.js App Router

### What is the App Router?
The App Router is Next.js's modern routing system that provides:
- Server-first routing with React Server Components
- Nested layouts and routes
- Simplified data fetching
- Built-in SEO optimizations
- Automatic code splitting

1. Create a new Next.js project:
```bash
npx create-next-app@latest my-nextjs-app
```

2. During setup, you'll see several prompts. Here are the options to select:
```
Would you like to use TypeScript? Yes
Would you like to use ESLint? Yes
Would you like to use Tailwind CSS? Yes
Would you like to use `src/` directory? No
Would you like to use App Router? Yes
Would you like to use the default import alias (@/*)? Yes
Would you like to use Turbopack? No
```

Note: We're using the default import alias (@/*) as it's the recommended configuration for Next.js. We're not using Turbopack as we want to ensure maximum stability for our development environment.

3. Navigate to your project directory and start the development server:
```bash
cd my-nextjs-app
npm run dev
```

### Project Structure Overview

In this guide, we'll be building a blog application with Next.js 15. Let's first understand the project structure we'll be creating:

```
my-nextjs-app/
├── app/                        # Main application directory
│   ├── lib/                    # Shared utility functions and data handling
│   │   ├── utils.ts           # General utility functions (formatting, validation, etc.)
│   │   └── data.ts           # Data fetching and API functions
│   ├── ui/                    # All UI components and styles
│   │   ├── components/        # Reusable UI components
│   │   │   ├── button.tsx    # Shared button component
│   │   │   └── card.tsx      # Blog post card component
│   ├── blog/                  # Blog feature directory
│   │   ├── [id]/             # Dynamic route for individual blog posts
│   │   │   └── page.tsx      # Individual blog post page
│   │   └── page.tsx          # Blog listing page
│   ├── globals.css           # Global styles and Tailwind CSS
│   ├── layout.tsx            # Root layout (applied to all pages)
│   └── page.tsx              # Home page of the application
├── public/                    # Static files directory
│   └── images/               # Image assets
└── next.config.js            # Next.js configuration file
```

Let's break down each directory and its purpose:

1. **/app Directory**
   - This is the heart of your application
   - Contains all routes, components, and application logic
   - Uses Next.js App Router for file-system based routing
   - Houses globals.css for application-wide styles

2. **/app/lib**
   - Houses shared utility functions and data handling logic
   - `utils.ts` contains helper functions like date formatting
   - `data.ts` manages data fetching and API interactions
   - Keeps your data layer separate from your UI layer

3. **/app/ui**
   - Contains all UI-related code and components
   - `/components` holds reusable UI elements
   - Promotes component reusability and consistent design

4. **/app/blog**
   - Feature-specific directory for blog functionality
   - Uses dynamic routing with `[id]` for individual posts
   - Demonstrates Next.js routing capabilities

5. **/public**
   - Stores static assets like images and fonts
   - Files here are served directly at the root URL
   - Optimized for static file serving

6. **Configuration Files**
   - `next.config.js` configures Next.js behavior
   - Handles things like environment variables and build options
   - Generally requires minimal modification for basic projects

## Part 2: Creating Your First Routes

1. First, let's create a data utility in `app/lib/data.ts`. This file will contain our data fetching logic and type definitions:

We place this in the `/lib` folder because it contains core functionality that can be reused across different parts of our application. The types and functions defined here will help us maintain consistency and type safety throughout our app:

```typescript
// app/lib/data.ts

// Define the shape of our blog post data
// This type ensures consistency across our application
export type BlogPost = {
  id: string;
  title: string;
  excerpt: string;
  content: string;
  views: number;
};

// This function simulates fetching blog posts from an API or database
// In a real application, this would connect to your data source
export const getBlogPosts = async (): Promise<BlogPost[]> => {
  // Simulate API delay
  await new Promise((resolve) => setTimeout(resolve, 1000));

  return [
    {
      id: "1",
      title: "Getting Started with Next.js 15",
      excerpt: "Learn the basics of Next.js and the App Router",
      content: "Full content would go here...",
      views: 100,
    },
    {
      id: "2",
      title: "Understanding Server Components",
      excerpt: "Deep dive into React Server Components",
      content: "Full content would go here...",
      views: 50,
    },
  ];
};

// Function to fetch a single blog post by ID
// Returns null if the post isn't found - this helps with error handling
export const getBlogPost = async (id: string): Promise<BlogPost | null> => {
  const posts = await getBlogPosts();
  return posts.find((post) => post.id === id) || null;
};

export async function getViewCount(postId: string): Promise<number> {
  const post = await getBlogPost(postId);
  return post?.views ?? 0;
}
```

2. Next, let's create a reusable card component in `app/ui/components/card.tsx`. This component will be used to display blog posts in a consistent format:

In Next.JS all components are SERVER components by default, meaning that the code to generate the UI runs on the server first and then only the completed markup is sent to the browser.

```typescript
// app/ui/components/card.tsx

// This is a reusable card component that accepts:
// - title: The card's header text
// - children: Any content to be displayed in the card body
// - className: Optional additional CSS classes for customization
export default function Card({
  title,
  children,
  className = '',
}: {
  title: string;
  children: React.ReactNode;
  className?: string;
}) {
  return (
    <div className={`rounded-lg border p-4 ${className}`}>
      <h2 className="text-xl font-semibold mb-2">{title}</h2>
      {children}
    </div>
  );
}
```

3. Now, let's create the blog listing page at `app/blog/page.tsx`. This Server Component introduces a fundamentally different approach to data fetching:

In Next.js, pages are Server Components by default, which means they run on the server. This enables us to fetch data directly in the component without useEffect. Here's why this is powerful:

1. **Data Fetching is Simpler:**
   - Instead of using useEffect and useState to manage data fetching
   - You can use async/await directly in your component
   - The data is fetched before the page is even sent to the browser
   - Next.js automatically handles loading states (we'll see this later with loading.tsx)

2. **How it Works:**
   - When a user visits the page, Next.js runs this component on the server
   - The server waits for getBlogPosts() to complete
   - The HTML is generated with all the blog posts already included
   - The final HTML is sent to the browser, ready to display
   - No extra loading spinners or content flashes!

3. **Why This is Better:**
   - Users see the content faster
   - Search engines can see all your content
   - Your database credentials stay safe on the server
   - Less code to write and maintain

Here's our implementation:

```typescript
// app/blog/page.tsx

// This is a Server Component (default in Next.js 15)
// Instead of useEffect, we can fetch data directly in our component!
import { getBlogPosts } from "@/app/lib/data";
import Card from "@/app/ui/components/card";
import Link from "next/link";

// Notice we can use 'async' directly on the component
// This means: wait for the data before sending HTML to the browser
export default async function BlogPage() {
  const posts = await getBlogPosts();
  
  return (
    <div className="max-w-4xl mx-auto p-4">
      <h1 className="text-2xl font-bold mb-6">Blog Posts</h1>
      
      <div className="space-y-4">
        {posts.map(post => (
          <Card 
            key={post.id}
            title={post.title}
            className="hover:border-blue-500 transition-colors"
          >
            <p className="text-gray-600 mb-4">{post.excerpt}</p>
            <Link
              href={`/blog/${post.id}`}
              className="text-blue-500 hover:text-blue-600"
            >
              Read more →
            </Link>
          </Card>
        ))}
      </div>
    </div>
  );
}
```

### Let's Test Our Blog Page

1. Make sure your development server is running:
```bash
npm run dev
```

2. Open your browser and navigate to: http://localhost:3000/blog

3. You should see:
   - A page with "Blog Posts" as the heading
   - Two blog post cards below
   - Each card should have a title, excerpt, and "Read more" link
   - The cards should have a subtle hover effect (border turns blue)

If you don't see the blog posts, check:
   - That your file is saved in the correct location: `app/blog/page.tsx`
   - That getBlogPosts is properly imported from '@/app/lib/data'
   - Your terminal for any error messages

Why this works:
- When you visit /blog, Next.js executes the Server Component
- The data is fetched server-side (notice there's no loading state!)
- The complete HTML is sent to your browser
- That's why you see the content immediately with no flashing or loading states

4. Next, let's create the dynamic blog post page at `app/blog/[id]/page.tsx`. This page will handle individual blog post views:

The [id] in the folder name creates a dynamic route segment in Next.js. This means it will match any value in that position of the URL and pass it to our page component as a parameter. For example, /blog/1 and /blog/2 will both use this page template:

```typescript
// app/blog/[id]/page.tsx
import { getBlogPost } from "@/app/lib/data";
import { notFound } from "next/navigation";

// Dynamic route page component for individual blog posts
// [id] in the folder name creates a dynamic route parameter
// params prop automatically receives the dynamic segment value from the URL
export default async function BlogPost({ params }: { params: { id: string } }) {
  // Extract the blog post ID from the URL parameters
  const { id } = await params;

  // Fetch the blog post data using the ID
  // This is an async operation that waits for the data
  const post = await getBlogPost(id);
  console.log(post);

  // If no post is found, trigger Next.js's not-found page
  if (!post) {
    console.log("No Post");
    notFound();
  }

  // Render the blog post with a responsive layout
  return (
    <article className="max-w-4xl mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">{post.title}</h1>
      <div className="prose lg:prose-xl">{post.content}</div>
    </article>
  );
}

```

### Let's Test Our Dynamic Routes

1. Make sure your development server is running:
```bash
npm run dev
```

2. Test the following URLs:
   - http://localhost:3000/blog/1
   - http://localhost:3000/blog/2

3. For each URL, you should see:
   - A full blog post page
   - The post title at the top
   - The full content below
   - Different content for each ID

4. Try an invalid ID (e.g., /blog/999):
   - You should see the 404 page (Next.js handles this automatically)
   - This happens because our getBlogPost function returns null for unknown IDs

Why this works:
- The [id] folder tells Next.js this is a dynamic route
- When you visit /blog/1, the '1' is passed to your component as params.id
- The server fetches just the data for that specific post
- You get a unique page for each blog post ID

## Part 3: Server vs Client Components

Let's explore both types of components by creating a views counter (Server Component) and a like button (Client Component):

1. First, let's create a views counter component that runs on the server:

```typescript
// app/ui/components/views-counter.tsx
import { getViewCount } from '@/app/lib/data';

export default async function ViewsCounter({ postId }: { postId: string }) {
  // This runs on the server - no useEffect needed!
  const views = await getViewCount(postId);
  
  return (
    <div className="text-sm text-gray-500">
      {views} views
    </div>
  );
}
```

2. Now, let's create an interactive like button that runs on the client:

```typescript
'use client';  // This marks the component as a Client Component

// app/ui/components/like-button.tsx
import { useState } from 'react';

export default function LikeButton() {
  // We can use hooks because this is a Client Component
  const [likes, setLikes] = useState(0);
  
  return (
    <button
      onClick={() => setLikes(prev => prev + 1)}
      className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
    >
      Likes: {likes}
    </button>
  );
}
```

3. Let's create a demo page to show both components working together:

```typescript
// app/demo/page.tsx
// Import both server and client components
import ViewsCounter from "@/app/ui/components/views-counter";
import LikeButton from "@/app/ui/components/like-button";

// This is a Server Component by default since it's in the app directory
export default function DemoPage() {
  return (
    <div className="max-w-4xl mx-auto p-4">
      <h1 className="text-2xl font-bold mb-6">
        Server vs Client Components Demo
      </h1>

      <div className="space-y-4 border rounded p-4">
        {/* ViewsCounter is a Server Component that fetches and displays data on the server */}
        <div>
          <h2 className="text-xl mb-2">Server Component:</h2>
          <ViewsCounter postId="1" />
        </div>

        {/* LikeButton is a Client Component (marked with 'use client') that handles interactivity */}
        <div>
          <h2 className="text-xl mb-2">Client Component:</h2>
          <LikeButton />
        </div>
      </div>
    </div>
  );
}
```

### Let's Test Our Components

1. Navigate to our demo page:
```bash
http://localhost:3000/demo
```

2. You should see:
   - The server-rendered view count (loads immediately)
   - The client-side like button (interactive)
   - The like count updates instantly when clicked

3. Try this experiment:
   - Open your browser's network tab (F12 > Network)
   - Refresh the page
   - Notice how the view count is already in the HTML
   - The like button functionality loads with the JavaScript

Why this works:
- The view counter comes pre-rendered from the server
- The like button hydrates on the client for interactivity
- This hybrid approach gives us the best of both worlds:
  * Fast initial page loads (server)
  * Interactive
 
### Creating Utility Functions

Let's create a utility function in `app/lib/utils.ts`. Utility functions are helper functions that can be used across your application:

This is a great example of code that can be shared between Server and Client Components. We place it in the lib folder because it's a pure function with no UI elements:

```typescript
// app/lib/utils.ts
export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  }).format(date);
}
```

## Part 4: Error and Loading States

### 1. Creating Loading States

Let's create a loading state for blog posts at `app/blog/loading.tsx`. Next.js will automatically show this loading UI when the page is loading:

Loading states are crucial for good UX. Next.js makes this easy with the special loading.tsx file that automatically creates a loading boundary. This file should be placed in the same directory as the page it's loading for:

```typescript
// app/blog/loading.tsx

// This loading component is shown while the blog page data is being fetched
// It provides a skeleton UI that matches the layout of the actual blog page
export default function Loading() {
  return (
    <div className="max-w-4xl mx-auto p-4">
      {/* Animated loading skeleton for the page title */}
      <div className="h-8 w-48 bg-gray-200 rounded mb-6 animate-pulse" />

      <div className="space-y-4">
        {/* Skeleton for a blog post card */}
        <div className="border rounded-lg p-4">
          {/* Animated loading skeleton for post title */}
          <div className="h-6 w-3/4 bg-gray-200 rounded animate-pulse mb-3" />
          {/* Animated loading skeleton for post excerpt */}
          <div className="h-4 w-1/2 bg-gray-200 rounded animate-pulse" />
        </div>
      </div>
    </div>
  );
}

```

### 2. Creating Error Handling

Let's create an error component at `app/blog/error.tsx`. This component will be shown automatically when an error occurs in your route:

Error components in Next.js 15 must be Client Components (notice the 'use client' directive) because they need to be interactive. They receive two props:
- error: The error object that was thrown
- reset: A function to attempt to recover from the error

Here's how we implement error handling:

```typescript
'use client';

// app/blog/error.tsx
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="max-w-4xl mx-auto p-4 text-center">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <button
        onClick={() => reset()}
        className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        Try again
      </button>
    </div>
  );
}
```

To test our error add the following line to blog/page.tsx

```typescript
export default async function BlogPage() {
  throw new Error("test"); // Add this line
```

This will cause the page to throw an error.

Navigate to /blog and notice we were routed to the error page.

## Part 5: Understanding Next.js 15 Features

Key concepts to remember:

1. **Project Structure**
   - `/app` - Routes and application logic
   - `/app/lib` - Utility functions and data fetching
   - `/app/ui` - UI components
   - `/public` - Static assets

2. **Server Components**
   - Default in Next.js
   - Better performance and SEO
   - Direct database/API access
   - No client-side JavaScript overhead

3. **Client Components**
   - Use 'use client' directive
   - Required for interactivity
   - Access to browser APIs and React hooks
   - Hydrated in the browser

4. **Special Files**
   - `layout.tsx` - Shared layouts
   - `loading.tsx` - Loading UI
   - `error.tsx` - Error boundaries
   - `not-found.tsx` - 404 pages

## Final Commit

1. Stage changes:
```bash
git add .
```

2. Create commit:
```bash
git commit -m "Guided Activity 2 Complete"
```

3. Push to GitHub:
```bash
git push
```

## Key Takeaways

1. Next.js Project Structure:
   - Organized, scalable folder structure
   - Clear separation of concerns
   - Improved developer experience

2. Server vs Client Components:
   - Server Components for better performance
   - Client Components only when needed
   - Automatic code splitting

3. Best Practices:
   - Keep data fetching in `/app/lib`
   - UI components in `/app/ui`
   - Use Server Components by default
   - Implement proper loading and error states

If you have any questions about this assignment, please reach out to your instructor or TA for this course.
