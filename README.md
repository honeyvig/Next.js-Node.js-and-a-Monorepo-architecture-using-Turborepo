# Next.js-Node.js-and-a-Monorepo-architecture-using-Turborepo
To build a full-stack application with Next.js, Node.js, and a Monorepo architecture using Turborepo, we'll break down the process into manageable steps and provide relevant code snippets. The goal is to structure a project where Next.js serves as the frontend and Node.js as the backend, with Turborepo managing the shared code and dependency graph.
1. Setting up a Monorepo with Turborepo

First, you’ll need to set up a monorepo using Turborepo. This allows you to organize multiple applications (e.g., Next.js frontend and Node.js backend) and share code between them.
1.1. Initialize the Monorepo

    Install Turborepo globally if you don’t have it installed:

npm install -g turbo

Create a new monorepo folder and initialize it:

mkdir my-monorepo
cd my-monorepo
npm init -y

Install Turborepo and other dependencies:

npm install --save-dev turbo

Create a turbo.json file to define your monorepo configuration:

    {
      "pipeline": {
        "build": {
          "dependsOn": []
        },
        "dev": {
          "dependsOn": ["build"]
        },
        "lint": {
          "dependsOn": ["build"]
        }
      }
    }

1.2. Create Next.js and Node.js Apps

In your monorepo, create two directories: one for the Next.js app and one for the Node.js API.

mkdir apps
cd apps
npx create-next-app frontend
mkdir backend

    frontend: This will be your Next.js application.
    backend: This will be your Node.js API.

Now, you have the structure:

/my-monorepo
  /apps
    /frontend (Next.js app)
    /backend (Node.js app)
  turbo.json

1.3. Add Shared Code (Optional)

To enable shared code between the apps (e.g., shared types or utility functions), you can create a packages directory.

mkdir packages

Now, you can add shared code in this directory and import it in both the frontend and backend.
2. Setting Up Next.js App

Next.js will serve as the frontend of your app. Here’s how to set up the frontend app.
2.1. Install Dependencies for Next.js

Go to your frontend directory and make sure you have everything set up for a Next.js application:

cd apps/frontend
npm install

2.2. Create a Simple Page in Next.js

In your pages/index.js, you can create a simple page that fetches data from your Node.js backend.

// apps/frontend/pages/index.js
import { useEffect, useState } from 'react';

const Home = () => {
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      const response = await fetch('/api/data'); // Calls backend API
      const result = await response.json();
      setData(result);
    };
    fetchData();
  }, []);

  return (
    <div>
      <h1>Welcome to Next.js</h1>
      {data ? (
        <pre>{JSON.stringify(data, null, 2)}</pre>
      ) : (
        <p>Loading...</p>
      )}
    </div>
  );
};

export default Home;

This example fetches data from your Node.js backend’s /api/data endpoint.
3. Setting Up Node.js App

The Node.js app will serve as the API for your frontend.
3.1. Install Dependencies for Node.js

Go to your backend directory and initialize a simple Express app.

cd apps/backend
npm init -y
npm install express

3.2. Create an API Endpoint

Create a simple API endpoint in apps/backend/index.js that sends some data to the frontend.

// apps/backend/index.js
const express = require('express');
const app = express();
const port = 3001;

// Middleware to handle CORS (if needed)
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
  next();
});

// Simple API endpoint
app.get('/api/data', (req, res) => {
  res.json({ message: 'Hello from Node.js backend!' });
});

app.listen(port, () => {
  console.log(`Backend listening at http://localhost:${port}`);
});

This sets up an API that returns a JSON object when the /api/data route is hit.
4. Connecting Frontend to Backend

In the Next.js app, you’re already calling the backend API at /api/data. Since both the frontend and backend are in the same monorepo, you’ll need to configure the frontend to call the backend during development.
4.1. Add Proxy for Local Development

To make it easier for the frontend to make requests to the backend while in development, you can configure a proxy in the Next.js app. You can use next.config.js to proxy API calls to the backend.

In apps/frontend/next.config.js, add the following:

module.exports = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'http://localhost:3001/api/:path*', // Proxy to backend
      },
    ];
  },
};

This will forward any request made to /api/* from the Next.js app to your Node.js backend running on port 3001.
5. Running the Monorepo

To run the project, you’ll need to start both the Next.js frontend and Node.js backend. You can use Turborepo to manage this easily.
5.1. Add Scripts to package.json

In the root package.json, add these scripts:

{
  "scripts": {
    "dev": "turbo run dev --parallel",
    "build": "turbo run build --parallel",
    "lint": "turbo run lint --parallel"
  }
}

Then, in each of the apps (frontend and backend), define their dev scripts:

    For frontend/package.json:

{
  "scripts": {
    "dev": "next dev"
  }
}

For backend/package.json:

    {
      "scripts": {
        "dev": "node index.js"
      }
    }

5.2. Start the App

Now, from the root of your monorepo, you can run the development environment for both apps:

npm run dev

Turborepo will run both the Next.js and Node.js apps in parallel, allowing you to work on both frontend and backend simultaneously.
6. Monorepo Best Practices

    Shared Code: If you have shared logic (like types or utility functions), you can place them in the packages folder. For example, create packages/utils for shared utilities and import them in both your frontend and backend.

    Example:

// packages/utils/index.js
export const greet = (name) => `Hello, ${name}`;

Then import it like this in your frontend and backend:

    import { greet } from '@my-monorepo/utils';

    Turborepo Pipeline: You can add tasks to the pipeline section of turbo.json to optimize builds, tests, and other tasks across the monorepo.

Conclusion

This setup uses Next.js for the frontend and Node.js for the backend, all managed within a Turborepo monorepo structure. With Turborepo, you can easily manage multiple apps in one codebase, allowing for a clean, scalable architecture. The basic code structure and configuration provided will help you set up the full-stack application, with shared code, easy development workflow, and the ability to scale as your project grows.
