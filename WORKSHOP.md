# Learn to build and develop a basic API project with Docker

This guide walks you through setting up an efficient local API development environment using **Docker**, **Node.js**, **TypeScript**, and **Express**.
It focuses on separating the build process for a smaller final image while maintaining hot-reloading for development.

---

## 1. Project setup and basic build

1. **Create project folder**  
   Create a new folder for your project in a sensible location, for example:

   ```shell
   mkdir -p ~/Documents/daemon-labs/docker-api
   ```

   > You can either create this via a terminal window or your file explorer.

2. **Open the new folder in your code editor**

   > If you are using VSCode, we can now do everything from within the code editor.

3. **Create `Dockerfile`**  
   Add the following content:

   ```Dockerfile
   FROM node:22-alpine

   WORKDIR /app
   ```

4. **Create `docker-compose.yaml`**  
   Add the following content to define your service:

   ```yaml
   ---
   services:
     app:
       build: .
   ```

5. **Initial image check**
   - Run the following command

     ```shell
     docker compose build
     ```

     > If you now run `docker images`, you'll see a newly created image which should be around 226MB in size.

   - Run the following command

     ```shell
     docker compose run --rm app node --version
     ```

     > The output should start with `v22` followed by the latest minor and patch version.

---

## 2. Dependency management and TypeScript config

1. **Initialise project and install dev dependencies**  
   We use explicit volume mounts (`-v ./app:/app`) in the following commands to ensure the generated files are saved back to your local host folder.
   - Run the following command

     ```shell
     docker compose run --rm -v ./app:/app app npm init -y
     ```

     > Notice how the `app` directory is automatically created on your host machine due to the volume mount.

   - Run the following command

     ```shell
     docker compose run --rm -v ./app:/app app npm add --save-dev @types/node@22 @tsconfig/recommended typescript
     ```

     > Notice this automatically creates a `package-lock.json` file.
     > Even though dependencies have been installed, if you run `docker images` again, you'll see the image size hasn't changed because the `node_modules` were written to your local volume, not the image layer.

2. **Create `tsconfig.json`**  
   Add the following content to configure the TypeScript compiler:

   ```json
   {
     "extends": "@tsconfig/recommended/tsconfig.json",
     "compilerOptions": {
       "outDir": "./dist"
     }
   }
   ```

   > ℹ️ While you could auto-generate this file, our manual configuration using a recommended preset keeps the file minimal and clean.

3. **Create source file and scripts**
   - Create `./src/index.ts` with the following:

     ```typescript
     console.log("Hello world!");
     ```

   - Add the following to the `scripts` section in your `package.json`:

     ```json
     "start": "node ./dist/index.js",
     "build": "tsc",
     ```

---

## 3. Production build and initial run

1. **Update `Dockerfile`**  
   Update the end of your `Dockerfile` to handle dependencies, build the project, and define the runtime command:

   ```Dockerfile
   COPY ./app/package*.json ./

   RUN npm ci

   COPY ./app .

   RUN npm run build

   CMD [ "npm", "start" ]
   ```

2. **Run final build**
   - Run the following command

     ```shell
     docker compose build
     ```

   - Run the following command

     ```shell
     docker compose run --rm app ls -la
     ```

     > Notice that we haven't included `-v ./app:/app` in this command, but the output still includes a `dist` folder. This is because the build step was executed _inside_ the image during the `docker compose build` process.

3. **Test the application**
   - Run the following command

     ```shell
     docker compose up
     ```

     > You should see a couple of lines of Node debug followed by your `Hello world!`.

---

## 4. Adding Express and port mapping

1. **Install production dependencies**
   - Run the following command

     ```shell
     docker compose run --rm -v ./app:/app app npm add express
     ```

     > This dependency is added to the `dependencies` section in your local `package.json`.

   - Run the following command

     ```shell
     docker compose run --rm -v ./app:/app app npm add --save-dev @types/express
     ```

2. **Update application code**  
   Update the `./src/index.ts` to the following Express server:

   ```typescript
   import express from "express";

   const app = express();
   const port = 3000;

   app.get("/", (_req, res) => res.json({ message: "Hello World!" }));

   app.listen(port, () => console.log(`Example app listening on port ${port}`));
   ```

3. **Rebuild and test**
   - Run the following command

     ```shell
     docker compose build
     ```

   - Run the following command

     ```shell
     docker compose up
     ```

   > ⚠️ The container is running but the port is not exposed.  
   > **Exit your container by pressing `Ctrl+C`** on your keyboard.

4. **Publish port**
   - Update `docker-compose.yaml` to include the port mapping:

     ```yaml
     ports:
       - 3000:3000
     ```

   - Run the following command

     ```shell
     docker compose up
     ```

     > If you open `http://localhost:3000` in your browser now, you should see `{"message":"Hello World!"}`.

---

## 5. Enabling hot reloading for development

1. **Prepare for live development**
   - Update the `./src/index.ts` from `Hello World!` to `Hello Universe!`

     > You'll notice this change is not reflected upon browser refresh, as the image still contains the old compiled code.

   - Update `docker-compose.yaml` to include the local volume mount for live syncing:

     ```yaml
     volumes:
       - ./app:/app
     ```

2. **Install development tools**
   - Run the following command

     ```shell
     docker compose run --rm app npm add --save-dev nodemon ts-node
     ```

     > Note how we no longer need the `-v ./app:/app` argument because the volume mount is now defined in the `docker-compose.yaml` file.

3. **Configure hot reloading**
   - Add a new script in `package.json` called `dev` using the robust command:

     ```json
     "dev": "nodemon --exec ts-node ./src/index.ts --legacy-watch",
     ```

   - Update `docker-compose.yaml` to override the default `CMD` with the new development command:

     ```yaml
     command:
       - npm
       - run
       - dev
     ```

4. **Final test**
   - Run the following command

     ```shell
     docker compose up
     ```

     > Your browser should now return the `Hello Universe!` message.

   - Update the `./src/index.ts` back to `Hello World!`
     > You'll notice in your container logs that `nodemon` has restarted due to changes, and your browser updates without requiring a manual stop/build/start cycle.

---

## 6. Improving image size with multi-stage builds

1. **Run a baseline build**
   - Run the following command:

     ```shell
     docker compose build
     ```

     > If you run `docker images` now, your image contains all dev dependencies and is about **334MB** in size.
     > We want to reduce this to only include what is needed for production.

2. **Create `.dockerignore`**  
   This prevents unnecessary files from being copied into the build context.

   ```dockerignore
   app/dist
   app/node_modules
   ```

   > We exclude auto-generated and local environment files to ensure clean, repeatable builds.

3. **Update the `Dockerfile` for multi-stage build**
   Replace the entire content of your `Dockerfile` with the following:

   ```Dockerfile
   FROM node:22-alpine AS base

   WORKDIR /app

   FROM base AS build

   COPY ./app/package*.json ./

   RUN npm ci

   COPY ./app .

   RUN npm run build

   FROM base

   COPY --from=build /app/package*.json ./
   COPY --from=build /app/dist ./dist

   RUN npm ci --only=production

   CMD [ "npm", "start" ]
   ```

4. **Create `docker-compose.base.yaml`**  
   This file defines the production-ready service configuration.

   ```yaml
   ---
   services:
     app:
       build: .
       ports:
         - 3000:3000
   ```

   > This gives us a base production setup.

5. **Update `docker-compose.yaml`**  
   This file now **extends** the base and adds the development-specific overrides (volumes and the `dev` command).

   ```yaml
   ---
   services:
     app:
       extends:
         file: docker-compose.base.yaml
         service: app
       command:
         - npm
         - run
         - dev
       volumes:
         - ./app:/app
   ```

6. **Run final build and check size**
   - Run the command:

     ```shell
     docker compose build
     ```

     > If you run `docker images` now, your final image should be significantly smaller (closer to **230MB**), as it only contains the production dependencies and the compiled code.

   - Test the production build:

     ```shell
     docker compose -f ./docker-compose.base.yaml up
     ```

     > This runs the application in production mode, using the `CMD [ "npm", "start" ]` from the new `Dockerfile`.

   - Test the development setup:

     ```shell
     docker compose up
     ```

     > This should still work exactly the same as it did before we made these changes.
