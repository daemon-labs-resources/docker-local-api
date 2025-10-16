# docker-local-api

This guide walks you through setting up a modern local API development environment using **Docker**, **Node.js**, **TypeScript**, and **Express**, complete with hot-reloading for an efficient workflow.

---

## Setup and Initialization

1.  **Create Project Folder**
    > If you are able to, create a Git repo, clone, and use that folder as your project root.

2.  **Create `Dockerfile`**
    Add the following content to define your base environment:

    ```Dockerfile
    FROM node:22-alpine

    WORKDIR /app
    ```

3.  **Create `docker-compose.yaml`**
    Add the following content to define your service and local volume:

    ```yaml
    ---
    services:
      app:
        build: .
        volumes:
          - ./app:/app
    ```

4.  **Install Dependencies**
    Execute the following commands to initialize your Node project and install core packages:

    -   **(Optional)** Run the following command

        ```shell
        docker compose build
        ```

    -   Run the following command

        ````shell
        docker compose run --rm app npm init -y
        ```
        > Notice how the `app` directory is automatically created on your host machine due to the volume mount.

    -   Run the following command
        ```shell
        docker compose run --rm app npm add --save-dev @types/node@22 @tsconfig/recommended typescript
        ```

---

## TypeScript Configuration

1.  **Create `tsconfig.json`**
    Add the following content to configure the TypeScript compiler:

    ```json
    {
      "extends": "@tsconfig/recommended/tsconfig.json",
      "compilerOptions": {
        "outDir": "./dist"
      }
    }
    ```

    > [!NOTE]
    > This configuration is manually created for brevity and clarity, utilizing a recommended preset.

2.  **Create `./src/index.ts`**
    Create the source file with a simple test:

    ```typescript
    console.log('Hello world!');
    ```

3.  **Add Build and Start Scripts**
    Add the following to the `scripts` section in your `package.json`:

    ```json
    "start": "node ./dist/index.js",
    "build": "tsc",
    ```

4.  **Test the Build Process**
    -   Run the following command

        ```shell
        docker compose run --rm app npm run build
        ```
        > You'll notice an `index.js` file has now appeared inside a newly created `./dist` folder.

    -   Run the following command

        ```shell
        docker compose run --rm app npm start
        ```
        > You should see a couple of lines of Node debug followed by your `Hello world!`.

---

## Adding Express and Port Mapping

1.  **Install Express**
    Install the core Express package and its TypeScript definitions:
    -   Run the following command

        ```shell
        docker compose run --rm app npm add express
        ```
        > Notice how this one is not marked as `--save-dev`. You'll see a new `dependencies` section has been added to `package.json`.

    -   Run the following command
        ```shell
        docker compose run --rm app npm add --save-dev @types/express
        ```

2.  **Update `./src/index.ts`**
    Replace the contents with the Express server code:

    ```typescript
    import express from 'express';

    const app = express();
    const port = 3000;

    app.get('/', (_req, res) => res.json({ message: 'Hello World!' } ));

    app.listen(port, () => console.log(`Example app listening on port ${port}`));
    ```

3.  **Test the Server (Initial)**
    -   Run the following command

        ```shell
        docker compose run --rm app npm run build
        ```

    -   Run the following command

        ```shell
        docker compose run --rm app npm start
        ```

    > [!IMPORTANT]
    > The container won't exit now, as the Express server is running. Accessing `http://localhost:3000` still won't work because the port isn't published yet. **Exit the container by pressing `Ctrl+C`** on your keyboard.

4.  **Set Default Command and Publish Port**
    -   Update the end of your `Dockerfile`:

        ```Dockerfile
        CMD [ "npm", "start" ]
        ```
        > We have now defined the default command for our container.

    -   Run the following command

        ```shell
        docker compose build
        ```

    -   Update `docker-compose.yaml` to include the port mapping:

        ```yaml
        ports:
          - 3000:3000
        ```

    -   Run the following command

        ```shell
        docker compose up
        ```
        > This command starts the service in detached mode if you add `-d`, but by default it runs in the foreground. If you open `http://localhost:3000` in your browser now, it should work.

---

## Enabling Hot Reloading (Final Step)

1.  **Install Development Tools**
    -   Update the `./src/index.ts` file from `Hello World!` to `Hello Universe!`
        > This change won't take effect until we implement hot-reloading.
    -   Run the following command
    
        ```shell
        docker compose run --rm app npm add --save-dev nodemon ts-node
        ```

2.  **Add Robust `dev` Script**
    Add a new script in `package.json` to start `nodemon` using the most reliable configuration:

    ```json
    "start": "node ./dist/index.js",
    "build": "tsc",
    "dev": "nodemon ./src/index.ts --legacy-watch",
    ```
    > [!TIP]
    > The flags `--legacy-watch` (to reliably detect file changes over Docker volumes, especially on Windows) are used to ensure the hot-reloading works for all users.

3.  **Switch to Development Command**
    Update `docker-compose.yaml` to override the default `CMD`:

    ```yaml
    command:
      - npm
      - run
      - dev
    ```

4.  **Final Test**
    -   Run the following command

        ```shell
        docker compose up
        ```
        > You should now see the `Hello Universe!` message in your browser.

    -   Update the `./src/index.ts` back to `Hello World!`
        > You'll immediately notice in your container logs that **`nodemon` has restarted** due to changes, and a refresh of your browser shows the updated content without having to manually stop and restart the container.
