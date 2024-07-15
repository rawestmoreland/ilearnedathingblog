---
title: "How to Create a CRUD App Using Next.js and Supabase"
seoTitle: "Next.js & Supabase CRUD App Guide"
seoDescription: "Learn how to build a CRUD app using Next.js and Supabase in this step-by-step tutorial ideal for beginners and experienced developers"
datePublished: Mon Jul 15 2024 12:00:50 GMT+0000 (Coordinated Universal Time)
cuid: clymxo1gb000r09l5hc8geohi
slug: how-to-create-a-crud-app-using-nextjs-and-supabase
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719754512769/0177e6ca-9368-46e5-8549-6888ee029eba.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1719754530968/d4be2615-254d-4529-90ab-fbacd2f56a36.png
tags: tutorial, typescript, crud, nextjs, supabase, crud-operations

---

In today's fast-paced web development world, creating full-stack applications quickly and efficiently is more important than ever. This is where Next.js and Supabase come into play, offering a powerful combination for building robust, scalable web applications with minimal backend configuration.

In this tutorial, we'll walk through the process of building a simple CRUD (Create, Read, Update, Delete) application using Next.js for our frontend and Supabase as our backend-as-a-service (BaaS) solution. By the end of this guide, you'll have a functional web app that allows you to manage a list of items, demonstrating the core operations of any data-driven application.

We'll cover:

* Setting up a Next.js project
    
* Creating a [Supabase](http://supabase.io) project and connecting it to our app
    
* Implementing each CRUD operation with Next.js and Supabase
    

Whether you're a seasoned developer looking to streamline your stack or a beginner eager to build your first full-stack application, this tutorial will provide you with the knowledge to get started with this powerful tech combo.

Let's dive in and start building!

## What is Supabase?

Supabase is an open-source Firebase alternative that provides a suite of tools for building scalable web applications. It offers a powerful combination of services that traditionally required juggling multiple providers or building from scratch. Here's what makes Supabase stand out:

1. **PostgreSQL Database**: At its core, Supabase provides a PostgreSQL database, offering the robustness and features of this popular relational database system.
    
2. **Real-time Subscriptions**: Built-in support for real-time data syncing, allowing your app to instantly react to database changes.
    
3. **Authentication**: A complete authentication system that supports email/password, magic links, and various OAuth providers.
    
4. **Auto-generated APIs**: Supabase automatically creates RESTful APIs for your database, saving you time on backend development.
    
5. **Storage**: Built-in storage solution for managing large files like images and videos.
    
6. **Edge Functions**: Serverless functions for running your backend code.
    

For our CRUD app, we'll primarily be using Supabase's database and auto-generated APIs. This will allow us to focus on building our Next.js frontend while Supabase handles the heavy lifting on the backend.

The combination of Next.js and Supabase is particularly powerful because it allows developers to build full-stack applications with JavaScript/TypeScript across the entire stack. This uniformity can significantly speed up development and reduce the complexity of your project.

## Setting Up Your Project

To get started with our CRUD app, we need to set up both our Next.js frontend and our Supabase backend. Let's tackle these one at a time.

### Creating a Next.js App

1. Open your terminal and run the following command:
    
    ```bash
    npx create-next-app@latest crud-app-nextjs-supabase
    ```
    
2. When prompted, choose the following options:
    
    * Would you like to use TypeScript? › Yes
        
    * Would you like to use ESLint? › Yes
        
    * Would you like to use Tailwind CSS? › Yes
        
    * Would you like to use `src/` directory? › No
        
    * Would you like to use App Router? › Yes
        
    * Would you like to customize the default import alias? › No
        
3. Once the installation is complete, navigate to your project directory:
    
    ```bash
    cd crud-app-nextjs-supabase
    ```
    

### Setting up a Supabase Project

1. Go to [supabase.com](https://supabase.com/) and sign up or log in.
    
2. Click on "New project" and give your project a name.
    
3. Choose a database password and your region. Note down the password for later use.
    
4. Click "Create new project" and wait for your project to be provisioned.
    
5. Once your project is ready, go to the "Settings" section in the sidebar, then "API".
    
6. You'll find your project URL and anon public API key here. We'll need these to connect our Next.js app to Supabase.
    

### Installing Supabase Client

Back in your terminal, install the Supabase client library:

```bash
npm install @supabase/supabase-js
```

With these steps completed, you now have a Next.js project set up and a Supabase project ready to use. In the next section, we'll connect these two pieces together.

## Connecting Next.js to Supabase

Now that we have both our Next.js app and Supabase project set up, it's time to connect them. We'll create a Supabase client that we can use throughout our application.

1. First, create a new file called `.env.local` in the root of your Next.js project. Add the following content, replacing the placeholder values with your actual Supabase project details:
    
    ```bash
    NEXT_PUBLIC_SUPABASE_URL=your-project-url
    NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
    ```
    
2. Next, create a new file `utils/supabase.ts` in your project directory. Add the following code:
    
    ```typescript
    import { createClient } from '@supabase/supabase-js'
    
    const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
    const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
    
    export const supabase = createClient(supabaseUrl, supabaseKey)
    ```
    
    This creates a Supabase client that we can import and use in our components.
    
3. Now, let's set up our database table. Go to your Supabase project dashboard, navigate to the "Table Editor" section, and create a new table called "items" with the following structure:
    
    * id: int8 (primary key, auto-increment)
        
    * created\_at: timestamptz (default: now())
        
    * name: text
        
    * description: text
        
4. To test our connection, let's modify the `app/page.tsx` file to fetch and display items from our Supabase database:
    
    ```typescript
    import { supabase } from '../utils/supabase'
    
    export default async function Home() {
      const { data: items, error } = await supabase
        .from('items')
        .select('*')
    
      if (error) {
        console.error('Error fetching items:', error)
      }
    
      return (
        <main className="flex min-h-screen flex-col items-center justify-between p-24">
          <h1 className="text-4xl font-bold mb-8">My Items</h1>
          <ul>
            {items?.map((item) => (
              <li key={item.id}>{item.name}</li>
            ))}
          </ul>
        </main>
      )
    }
    ```
    
5. Start your Next.js development server:
    
    ```bash
    npm run dev
    ```
    

Visit [`http://localhost:3000`](http://localhost:3000) in your browser. You should see a page with the title "My Items" and an empty list (assuming you haven't added any items to your database yet).

Congratulations! You've successfully connected your Next.js app to Supabase. In the next section, we'll implement the full CRUD functionality.

Excellent. Let's move on to implementing the CRUD (Create, Read, Update, Delete) operations. We'll cover each operation one by one, keeping our explanations concise but informative.

---

## Implementing CRUD Operations

![Mad Edward Nygma GIF by Gotham](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExYm9xN3RmbHk1aTdwaGx6NmpkZXBjc3FwdHc2MDJhanl5MXdpczV4dSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/l3V0mgFspVuDAJK9y/giphy.gif align="center")

Now that our Next.js app is connected to Supabase, let's implement the four CRUD operations: Create, Read, Update, and Delete.

### Create: Adding New Items

Let's add a form to create new items:

1. Update `app/page.tsx`:
    

```typescript
'use client'
import { useState } from 'react'
import { supabase } from '../utils/supabase'

export default function Home() {
  const [items, setItems] = useState([])
  const [newItemName, setNewItemName] = useState('')
  const [newItemDescription, setNewItemDescription] = useState('')

  const fetchItems = async () => {
    const { data, error } = await supabase.from('items').select('*')
    if (error) console.error('Error fetching items:', error)
    else setItems(data)
  }

  const addItem = async (e) => {
    e.preventDefault()
    const { data, error } = await supabase
      .from('items')
      .insert([{ name: newItemName, description: newItemDescription }])
      .select()
    if (error) console.error('Error adding item:', error)
    else {
      setNewItemName('')
      setNewItemDescription('')
      fetchItems()
    }
  }

  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <h1 className="text-4xl font-bold mb-8">My Items</h1>
      
      <form onSubmit={addItem} className="mb-8">
        <input
          type="text"
          value={newItemName}
          onChange={(e) => setNewItemName(e.target.value)}
          placeholder="Item name"
          className="mr-2 p-2 border rounded"
        />
        <input
          type="text"
          value={newItemDescription}
          onChange={(e) => setNewItemDescription(e.target.value)}
          placeholder="Item description"
          className="mr-2 p-2 border rounded"
        />
        <button type="submit" className="p-2 bg-blue-500 text-white rounded">Add Item</button>
      </form>

      <ul>
        {items.map((item) => (
          <li key={item.id}>{item.name}: {item.description}</li>
        ))}
      </ul>
    </main>
  )
}
```

### Read: Fetching and Displaying Items

We've already implemented the read operation in the code above. The `fetchItems` function retrieves all items from the database, and we display them in an unordered list.

### Update: Modifying Existing Items

Let's add the ability to update items:

```typescript
// Add this function inside the Home component
const updateItem = async (id, newName, newDescription) => {
  const { error } = await supabase
    .from('items')
    .update({ name: newName, description: newDescription })
    .eq('id', id)
  if (error) console.error('Error updating item:', error)
  else fetchItems()
}

// Modify the list rendering to include an update button
<ul>
  {items.map((item) => (
    <li key={item.id}>
      {item.name}: {item.description}
      <button onClick={() => updateItem(item.id, prompt('New name'), prompt('New description'))}>
        Update
      </button>
    </li>
  ))}
</ul>
```

### Delete: Removing Items

Finally, let's add the ability to delete items:

```typescript
// Add this function inside the Home component
const deleteItem = async (id) => {
  const { error } = await supabase
    .from('items')
    .delete()
    .eq('id', id)
  if (error) console.error('Error deleting item:', error)
  else fetchItems()
}

// Modify the list rendering to include a delete button
<ul>
  {items.map((item) => (
    <li key={item.id}>
      {item.name}: {item.description}
      <button onClick={() => updateItem(item.id, prompt('New name'), prompt('New description'))}>
        Update
      </button>
      <button onClick={() => deleteItem(item.id)}>Delete</button>
    </li>
  ))}
</ul>
```

With these changes, you now have a fully functional CRUD application using Next.js and Supabase!

## Conclusion

Congratulations! You've successfully built a basic CRUD (Create, Read, Update, Delete) application using Next.js and Supabase. Let's recap what we've accomplished:

1. Set up a Next.js project and a Supabase backend
    
2. Connected our Next.js app to Supabase
    
3. Implemented all four CRUD operations:
    
    * Created new items
        
    * Read and displayed items from the database
        
    * Updated existing items
        
    * Deleted items from the database
        

This simple application demonstrates the power and simplicity of combining Next.js with Supabase. With just a few lines of code, we've created a fully functional web application with a robust backend.

### Next Steps

To further enhance your application, consider exploring these areas:

1. **Implement authentication**: Supabase offers easy-to-use authentication. Try adding user sign-up and login to your app.
    
2. **Improve the UI**: Use Tailwind CSS or another styling solution to make your app more visually appealing.
    
3. **Add real-time functionality**: Supabase supports real-time subscriptions. Try updating your list of items in real-time when changes occur.
    
4. **Server-side rendering**: Explore Next.js's server-side rendering capabilities to improve your app's performance and SEO.
    
5. **Error handling**: Implement more robust error handling and user feedback for a better user experience.
    

Remember, this is just the beginning. The combination of Next.js and Supabase provides a powerful foundation for building complex, scalable web applications. Keep exploring, and happy coding!