# NxAwesomeTodos

Awesome project to learn how to setup a fullstack project with Nx

Tech stack

- TypeScript
- Next.js
- Express.js
- React components
- Storybook

## Setup

### Create the project

```sh
yarn create nx-workspace
```

Answer this to questions

- Workspace name > nx-awesome-todos
- What to create in the new workspace > express
- Application name > api
- Use Nx Cloud? > No

### Setup API project

Create file `apps/api/src/app/index.ts` and paste the following content

```ts
import * as express from 'express';

const todos = [
  {
    id: 1,
    content: 'First todo',
    completed: false,
  },
  {
    id: 2,
    content: 'Second todo',
    completed: true,
  },
];

const app = express();

app.get('/api/todos', (req, res) => {
  res.send({ todos });
});

export default app;
```

Replace the content of `apps/api/src/main.ts` with

```ts
import app from './app';

const port = process.env.port || 3333;

const server = app.listen(port, () => {
  console.log(`Listening at http://localhost:${port}/api`);
});

server.on('error', console.error);
```

Run server

```sh
nx serve api
```

Check that everything is working making a GET request to http://localhost:3333/api/todos

### Setup API tests

Install supertest

```sh
yarn add supertest
```

Create the first test at `apps/api/test/api.test.ts`

```ts
import * as supertest from 'supertest';
import app from '../src/app';

const requestWithSupertest = supertest(app);

describe('Example describe', () => {
  test('Example test', () => {
    expect(1).toBe(1);
  });
});

describe('Generic endpoints', () => {
  test('GET /api/todos', async () => {
    const res = await requestWithSupertest.get('/api/todos');
    expect(res.status).toEqual(200);
  });
});
```

Test the app

```sh
nx test api
```

### Setup common library

```sh
nx g @nrwl/node:lib shared-types
```

Add an interface at `libs/shared-types/src/lib/shared-types.ts`

```ts
export interface Todo {
  id: number;
  content: string;
  completed: boolean;
}
```

Import the interface at `apps/api/src/app/index.ts`

```ts
import { Todo } from '@nx-awesome-todos/shared-types';
import * as express from 'express';

const todos: Todo[] = [
  // Rest of the code
];
```

### Setup client

This command create the app and the asociate Cypress setup

```sh
 nx g @nrwl/next:app client
```

#### Setup CORS

Enable CORS at server

```sh
yarn add cors
yarn add -D @types/cors
```

Setup cors at server `apps/api/src/main.ts`

```ts
import * as cors from 'cors';

app.use(cors());
```

#### Get data from server

Create a file at `apps/client/pages/api/todos.ts` with this content

```ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { Todo } from '@nx-awesome-todos/shared-types';

type Data = Todo[];

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<Data>
) {
  const response = await fetch('http://localhost:3333/api/todos');
  const todos: Todo[] = await response.json();
  res.status(200).json(todos);
}
```

Change the content of `apps/client/pages/index.tsx`

```tsx
import { useEffect, useState } from 'react';
import { Todo } from '@nx-awesome-todos/shared-types';
import styles from './index.module.scss';

export function Index() {
  const [todos, setTodos] = useState<Todo[]>([]);

  useEffect(() => {
    fetch('/api/todos')
      .then((res) => res.json())
      .then((res) => setTodos(res.todos));
  }, []);

  return (
    <div className={styles.page}>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            {todo.content} - {todo.completed ? 'completed' : 'not completed'}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default Index;
```
### Setup client styles
Creat global variables and styles. Create a folder named `styles` and a file named `variables.scss`
```scss
$test-gray: gray;
```
Load style variables
```js
// eslint-disable-next-line @typescript-eslint/no-var-requires
const withNx = require('@nrwl/next/plugins/with-nx');
const path = require('path');

/**
 * @type {import('@nrwl/next/plugins/with-nx').WithNxOptions}
 **/
const nextConfig = {
  nx: {
    // Set this to true if you would like to to use SVGR
    // See: https://github.com/gregberge/svgr
    svgr: false,
  },
  reactStrictMode: true,
  sassOptions: {
    includePaths: [path.join(__dirname, 'styles')],
    prependData: `@import "variables.scss";`,
  },
  images: {
    domains: ['images.unsplash.com'],
  },
};

module.exports = withNx(nextConfig);
```

Restart the development server for the changes to take effect.
### Setup client eslint

### Setup client stylelint

### Setup client tests
