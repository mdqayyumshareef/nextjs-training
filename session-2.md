# Welcome to the session

Let's get our hands dirty and build a blog with Next.js! After all, the best way to learn is by doing... and maybe a bit of trial and error.

It will be fun, I promise (or at least I hope so) 😅

## Project Setup

🛠️ create a new folder as "nextjs-blog" and open it in VSCode.

🛠️ open the terminal and run `npx create-next-app@latest`

You will be asked a few questions just reply them as below 👇

Would you like to use TypeScript? > Yes\
Would you like to use ESLint? > Yes\
Would you like to use Tailwind CSS? > Yes\
Would you like your code inside a `src/` directory? > Yes\
Would you like to use App Router? (recommended) > Yes\
Would you like to use Turbopack for `next dev`? > No\
Would you like to customize the import alias (`@/*` by default)? > No

🛠️ Install the node_modules using `npm i`

🛠️ Run the app using `npm run dev`

Let's open your favorite browser and enter [localhost:3000](http://localhost:3000) into the address bar to see the default page of the next.js app.

Congratulations! You have just created a brand new Next.js app! 😉

## Project Structure

### Top level folders

used to organize the application code and static assets

`app` - App router\
`public` - Static assets to be served\
`src` - Application source folder

### Top level files

used to configure your application, manage dependencies, run middleware, integrate monitoring tools, and define environment variables.

`package.json` - Project dependencies and scripts\
`.next.config.js` - Configuration file for next.js\
`tsconfig.json` - Configuration file for Typescript\
`tailwind.config.js` - Configuration file for TailwindCSS\
`.eslintrc.json` - Configuration file for ESLint\
`.env` - Environment variables\
`middleware.ts` - Next.js request middleware

### Routes

`folder` - Route segment\
`folder/folder` - Nested route segment\
`[folder]` - Dynamic route segment

### Routing files

`page` - Page file is used to define a page\
`layout` - Layout file is used to define a layout which is shared among pages\
`loading` - Loading file creates a loading state build on Suspense\
`error` - Error file allows us to handle runtime error and display a fallback UI\
`global-error` - Located in the root app directory, allows us to handle errors in root layout

Apart from the things mentioned here, there is whole universe of information just a [click away.](https://nextjs.org/docs/app/getting-started/project-structure#routing-files) You are welcome! 😉
🚏

## First Route

Before we crate our first route lets delete everything inside of `globals.css` file and copy, paste below code.

```CSS
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: Arial, Helvetica, sans-serif;
}
```

Alright, time to roll up our sleeves and craft a shiny ✨ new route 🚏

🛠️ Create a new folder in `app` directory and name it after your favorite movie character while following kebab-case naming convention. I am going with `john-wick`🕵️‍♂️

🛠️ Create `page.tsx` file inside the new folder.

🛠️ Copy and paste the blow code. Save the changes.

```jsx
import React from "react";

export default function JohnWickPage() {
  return (
    <div>
      <p>Hi I am John Wick</p>
      <p>My favorite quote is</p>
      <p>-- A man without purpose is nothing</p>
    </div>
  );
}
```

Hope the dev server is still running along!, if not, no worries it does not mean you are not smart. Just kidding! 😄 Simply run `npm run dev` in the terminal, and boom! you are back in action! 🚀

Alright let's head over to the browser, simple type the path manually, for my character it's [http://localhost/john-wick](http://localhost:3000/john-wick) and just like that, your new route is ready 🤓

### A Closer Look At Rendering

Alright, folks, it's time to summon the superhero of debugging the legendary `console.log()`

🦸‍♂️✨ Let's see it in action while we unravel the mysteries of rendering.

🛠️ Jump into the character page and add a log statement as `console.log("Hi Mom")`

🛠️ Open the browser's developer tools and check the console.

That's it, you are good to go, but wait... what's that little `server` tag before the log?

#### What's Happening Here?

By default, Next.js renders every page on the server side. That's why the log has the server tag.

#### When Do We Need Client-Side Rendering?

If you are creating an interactive UI or may be want to access browser APIs then it's time to move to client side rendering. Let's do it.

🛠️ Add the `use client` directive at the very top of your file, before anything else -yes, even before the imports.

🛠️ Head back to the browser, refresh, and check the console again.

Notice something different? That sneaky `server` tag is gone! 🕵️‍♂️ Your page is now rizzing with the browser. 💞

And there you have it, a seamless transformation from server-side to client-side rendering, with a pinch of debugging magic. 🪄

## Shadcn/UI Setup

[Shandcn](https://ui.shadcn.com/) offers a collection of re-usable components that we can use in our web applications. Follow the below command to setup the library.

🛠️ Run `npx shadcn@latest init` in the terminal.

You will be asked a few questions, just reply them as below 👇

Which style would you like to use? > New York/
Which color would you like to use as the base color? › Neutral\
Would you like to use CSS variables for theming? > yes\
How would you like to proceed? > Use --force

🛠️ Give the following commands in the terminal sequentially to fetch the components from Shadcn.

* `npx shadcn@latest add button`
* `npx shadcn@latest add card`
* `npx shadcn@latest add input`
* `npx shadcn@latest add label`

Whenever you pull a component from Shadcn, it gets teleported straight to the `src/components/ui` folder. Think of it as the VIP lounge for the components. 🛋️ Go on, check it out.

Apart from Shadcn, you can create your own custom components. Let's create one for our use.

🛠️ Create a new folder as `custom` inside `src/components`

🛠️ Create a new file, name it as `container.tsx` then copy, paste the blow code

```jsx
import { cn } from "@/lib/utils";
import React, { HTMLAttributes } from "react";

interface Props extends HTMLAttributes<HTMLDivElement> {
  className?: string;
}

export const Container = ({ className, children, ...props }: Props) => (
  <div className={cn("container pt-4", className)} {...props}>
    {children}
  </div>
);
```

## Register User Page

Alright, Let's get back to our actual blog! So we need a user registration page. After all, how else they join the club? 😅

🛠️ Let's bring in [react-hook-form](https://www.npmjs.com/package/react-hook-form) and [zod](https://www.npmjs.com/package/zod) by running `npm i react-hook-form zod @hookform/resolvers`

🛠️ Create a folder in `app` directory as `register`.

🛠️ Create a `page.tsx` file inside `register` folder, copy & paste the below code.

```jsx
"use client";

import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Container } from "@/components/custom/container";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { zodResolver } from "@hookform/resolvers/zod";
import Link from "next/link";
import { useForm } from "react-hook-form";
import { registerSchema } from "./schema";

export default function RegisterPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: zodResolver(registerSchema),
    mode: "onBlur",
  });

  const formSubmit = (data) => {
    console.log(data);
  };

  return (
    <Container>
      <Card>
        <CardHeader>
          <CardTitle>Lets Register Account</CardTitle>
          <CardDescription>
            Hello 👋, You have a grateful journey
          </CardDescription>
        </CardHeader>
        <form onSubmit={handleSubmit(formSubmit)}>
          <CardContent>
            <div className="grid grid-cols-2 gap-x-4 gap-y-2">
              <div className="">
                <Label htmlFor="first-name" className="inline-block mb-2">
                  First Name
                </Label>
                <Input id="first-name" {...register("firstName")} />
                {errors.firstName?.message && (
                  <span className="text-xs text-red-500">
                    {errors.firstName.message?.toString()}
                  </span>
                )}
              </div>
              <div className="">
                <Label htmlFor="last-name" className="inline-block mb-2">
                  Last Name
                </Label>
                <Input id="last-name" {...register("lastName")} />
                {errors.lastName?.message && (
                  <span className="text-xs text-red-500">
                    {errors.lastName.message?.toString()}
                  </span>
                )}
              </div>
              <div className="">
                <Label htmlFor="email" className="inline-block mb-2">
                  Email
                </Label>
                <Input id="email" {...register("email")} />
                {errors.email?.message && (
                  <span className="text-xs text-red-500">
                    {errors.email.message?.toString()}
                  </span>
                )}
              </div>
              <div className="">
                <Label htmlFor="password" className="inline-block mb-2">
                  Password
                </Label>
                <Input
                  id="password"
                  type="password"
                  {...register("password")}
                />
                {errors.password?.message && (
                  <span className="text-xs text-red-500">
                    {errors.password.message?.toString()}
                  </span>
                )}
              </div>
            </div>
          </CardContent>
          <CardFooter className="flex flex-col items-start">
            <Button type="submit">Submit</Button>
            <p className="mt-2 text-sm">
              Already have an account?
              <Link
                href={"/login"}
                className="ml-1 text-sm text-blue-600 hover:text-blue-700 hover:underline"
              >
                Login
              </Link>
            </p>
          </CardFooter>
        </form>
      </Card>
    </Container>
  );
}
```

🛠️ Create another file as `schema.tsx` inside `register` folder, copy & paste the below code.

```jsx
import { z } from "zod";

export const registerSchema = z.object({
  firstName: z.string().min(1, "Required"),
  lastName: z.string().min(1, "Required"),
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "Password should have minimum 6 characters"),
});
```

If you here then save all the changes, and run the dev server. Navigate to [http://localhost:3000/register](http://localhost:3000/register) to see the register user page.

Avoid clicking on the login link, it's like ringing the doorbell of a house that has not been built yet.

There is a slight UI issue, our page container does not seem to be in center. let's fix it.

🛠️ Open the `tailwind.config.ts` file, copy & paste the below code inside `theme` object.

```json
 container: {
  center: true,
  padding: "2rem",
  screens: {
    "2xl": "1400px",
  },
},
```

Save those changes and check the register page in your browser. The container? It is exactly in the center, as if TailwindCSS finally decided to cooperate for once.

Now, time for some action. Fill out the register form (don’t just type 'asdf', I see you), click submit, and check the console. Your input values will be there, standing at attention, reporting for duty!

If they are missing, it’s probably because the Console High Table demand a second look at your code. 😅

## Database Setup

To keep our data from ghosting us every time we refresh, we need a database. That's where SQLite comes in. For this project, we will use it to keep our data safe, sound, and ready for action!

🛠️ Open the terminal and run `npm i sequelize sqlite3` then run `npm i -D @types/sequelize` to install the types.

🛠️ Create a new folder as `db` inside `src` directory.

🛠️ Create a file as `index.ts` inside `db` folder, the copy & paste the below code

```ts
import { Sequelize } from "sequelize";
import sqlite3 from "sqlite3";

export const sequelize = new Sequelize({
  dialect: "sqlite",
  dialectModule: sqlite3,
  storage: "./blog.sqlite",
});

sequelize
  .sync({ alter: true })
  .then(() => {
    console.log("Database has been synced");
  })
  .catch((error) => {
    console.error("Unable to sync database", error);
  });

sequelize
  .authenticate()
  .then(() => {
    console.log("Connection has been established successfully.");
  })
  .catch((error) => {
    console.error("Unable to connect to the database:", error);
  });
```

Let’s see if SQLite is ready to roll!

🛠️ Open your terminal and type `tsx src/db/index.ts` it’s like knocking ✊ on your database's door 🚪 to say, 'Hello, are you there?'

If `tsx` is not installed, fix that with a simple `npm i -g tsx` It’s basically the command-line butler for your TypeScript files. 😅

After running the command, you will notice a new file named `blog.sqlite` in the root level. That’s where our data will live, rent-free. Treat it well it’s doing all the heavy lifting 🏋️ for us!

### User Model

🛠️ Create a new folder as `models` inside the `db` directory.

🛠️ Create a new file as `user.ts` inside the `models` folder, then copy & paste the below code.

```ts
import { DataTypes } from "sequelize";
import { sequelize } from "..";

export const User = sequelize.define("Users", {
  id: {
    type: DataTypes.UUID,
    defaultValue: DataTypes.UUIDV4,
    primaryKey: true,
  },
  firstName: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  lastName: {
    type: DataTypes.STRING,
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false,
  },
});
```

## Register Server Action

Since we are handling user passwords, encryption is a must. So, let’s scramble them up real good, like an omelet only the app can understand!

🛠️ Open the terminal and install bcrypt and it's types by running below commands.

* `npm i bcrypt`
* `npm i -D @types/bcrypt`

🛠️ Create a new file as `action.ts` inside `register` directory, then Copy & paste the below code

```jsx
"use server";

import { User } from "@/db/models/user";
import bcrypt from "bcrypt";
import { redirect } from "next/navigation";
import { z } from "zod";
import { registerSchema } from "./schema";

export async function RegisterAction(data: z.infer<typeof registerSchema>) {
  const saltRounds = 10;
  const salt = await bcrypt.genSalt(saltRounds);
  const { password, ...rest } = data;
  const passwordHash = await bcrypt.hash(password, salt);
  const user = { ...rest, password: passwordHash };
  await User.create(user);
  redirect("/login");
}
```

🛠️ Alright, import the `RegisterAction` function into the register page, because no code can survive without it (dramatic, but true). Then call it in the formSubmit function like below

```jsx
const formSubmit = (data) => {
  RegisterAction(data);
};
```

Pro tip: Forgetting to import `RegisterAction` is a rite of passage we have all been through. But let’s skip the 'Why isn’t it working?!' panic attack this time, okay? 😜

## Login Page

🛠️ Create a new folder as `login` inside `app` directory.

🛠️ Create a `page.tsx` file inside `login` folder, then copy & paste the below code.

```jsx
import { Container } from "@/components/custom/container";
import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import Link from "next/link";

export default function LoginPage() {
  return (
    <Container>
      <Card>
        <CardHeader>
          <CardTitle>Login</CardTitle>
        </CardHeader>
        <form>
          <CardContent>
            <div className="">
              <Label htmlFor="email" className="inline-block mb-2">
                Email
              </Label>
              <Input
                id="email"
                name="email"
                placeholder="Enter your email"
                required
              />
            </div>
            <div className="">
              <Label htmlFor="password" className="inline-block mb-2">
                Password
              </Label>
              <Input
                id="password"
                type="password"
                name="password"
                placeholder="Enter your password"
                required
              />
            </div>
          </CardContent>
          <CardFooter className="flex flex-col items-start">
            <Button type="submit">Submit</Button>
            <p className="mt-2 text-sm">
              Don't have an account?
              <Link
                href={"/register"}
                className="ml-1 text-sm text-blue-600 hover:text-blue-700 hover:underline"
              >
                Register
              </Link>
            </p>
          </CardFooter>
        </form>
      </Card>
    </Container>
  );
}
```

Alright, time to shine!

Save all the files (don't be that person who wonders why their changes didn’t apply 😜) and let’s fire up the app with a quick `npm run dev`

Next, head to the register page (no fake names like 'John Doe' this time, okay?) fill in the details, and BAM after a bit of SQLite magic, you will be redirected to the login page faster than you can say 'JSON.stringify'. 😃

That’s it for today! Until next time, may your bugs be few and your commits be descriptive!" 🙌
