# Welcome to the session

Let's roll up our sleeves and continue working on the blog project.

## Duplicate Email Bug Fix

Did you notice we can register multiple times with the same email? Don’t worry if you missed it. It’s a sneaky bug! Let’s squash it.

🛠️ Open the user model, update the email attribute as below.

```js
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
  }
```

🛠️ In the user model copy & paste the below code. we will remove this code after dropping the user table.

```js
User.drop()
  .then(() => {
    console.log("Dropped the user table");
  })
  .catch((error) => {
    console.error("unable to drop user table", error);
  });
```

🛠️ In the terminal execute `tsx src/db/models/user.ts` to drop the user table.

Don't forget to remove the `User.drop()` function, you can comment out as well.

🛠️ Update the `RegisterAction`.

```js
"use server";

import { User } from "@/db/models/user";
import bcrypt from "bcrypt";
import { redirect } from "next/navigation";
import { z } from "zod";
import { registerSchema } from "./schema";

export async function RegisterAction(
  prevState: unknown,
  data: z.infer<typeof registerSchema>
) {
  try {
    const saltRounds = 10;
    const salt = await bcrypt.genSalt(saltRounds);
    const { password, ...rest } = data;
    const passwordHash = await bcrypt.hash(password, salt);
    const user = { ...rest, password: passwordHash };
    await User.create({ ...user });
    redirect("/login");
  } catch (e) {
    if (e?.name === "SequelizeUniqueConstraintError") {
      return { message: "Email already exists" };
    }
    throw e;
  }
}
```

🛠️ Update the register page.

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
import { RegisterAction } from "./action";
import { startTransition, useActionState } from "react";

export default function RegisterPage() {
  const [state, action, isPending] = useActionState(RegisterAction, undefined);
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: zodResolver(registerSchema),
    mode: "onBlur",
  });

  const formSubmit = (data) => {
    startTransition(() => action(data));
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
                {state?.message && (
                  <p className=" text-red-400 text-sm  font-medium">
                    {state?.message}
                  </p>
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
            <Button type="submit" disabled={isPending}>
              Submit
            </Button>
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

Let's do some testing, save everything and make sure project is running before heading over to the register page. Fill in the form with email which already exists in the database.

You should see an error message that we are returning from the RegisterAction.

## Blog Model

Lets create a schema for saving blogs in the database.

🛠️ Create a new file as `blog.ts` inside `src/db/models` folder, then copy & paste the below code.

```ts
import { DataTypes, Model, Optional } from "sequelize";
import { sequelize } from "..";
import { User, UserInstance } from "./user";

interface BlogAttributes {
  id: string;
  title: string;
  content: string;
  userId: string;
}

type BlogCreationAttributes = Optional<BlogAttributes, "id">;

interface BlogInstance
  extends Model<BlogAttributes, BlogCreationAttributes>,
    BlogAttributes {
  createdAt?: Date;
  updatedAt?: Date;
  User?: UserInstance;
}

export const Blog = sequelize.define<BlogInstance>("Blogs", {
  id: {
    type: DataTypes.UUID,
    defaultValue: DataTypes.UUIDV4,
    primaryKey: true,
  },
  title: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  content: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  userId: {
    type: DataTypes.UUID,
    allowNull: false,
    references: {
      model: User,
      key: "id",
    },
  },
});

User.hasMany(Blog, { foreignKey: { name: "userId" } });
Blog.belongsTo(User, { foreignKey: { name: "userId" } });
```

🛠️ If you are getting import error for `UserInstance`, just open the user model and export the `UserInstance`.

🛠️ Sync the database with the Blog model by running `tsx src/db/index.ts` in the terminal.

The relationship between the Blog and the User is a classic one-to-many setup. Think of it like this: a user can write multiple blogs, but each blog belongs to just one user.

If you are curious, take a moment to explore how to create other types of associations using Sequelize ORM! [here](https://sequelize.org/docs/v6/core-concepts/assocs/).

## Create Blog Action

🛠️ Create a new file as `action.ts` inside `src/app/blog/new` folder, then copy & paste the below code.

```jsx
"use server";

import { Blog } from "@/db/models/blog";
import { getSession } from "@/lib/session";
import { redirect } from "next/navigation";

type Params = {
  title: string;
  content: string;
};

export async function CreateBlogAction(data: Params) {
  if (!data.title || !data.content) {
    throw new Error("Missing blog data.");
  }
  const token = await getSession();
  const userId = token?.userDetails?.id;
  await Blog.create({
    title: data.title,
    content: data.content,
    userId: userId!,
  });
  redirect("/");
}
```

Let's link the CreateBlogAction to the blog writing page.

🛠️ Open the blog writing page `src/app/blog/new/page.ts` copy & paste the below code.

```jsx
"use client";

import { Container } from "@/components/custom/container";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { useState } from "react";
import { CreateBlogAction } from "./action";
import dynamic from "next/dynamic";

const Editor = dynamic(
  () => import("@/components/custom/editor").then((x) => x.Editor),
  {
    ssr: false,
    loading: () => <p>Loading...</p>,
  }
);

export default function Blog() {
  const [title, setTitle] = useState("");
  const [editorData, setEditorData] = useState("");

  const handlePublish = () => {
    CreateBlogAction({ title, content: editorData });
  };

  return (
    <Container>
      <h1 className="mb-8">From Thoughts to Words: Blog Without Limits</h1>
      <Label htmlFor="title" className="inline-block mb-2">
        Title
      </Label>
      <Input
        id="title"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
      />
      <div className="my-4">
        <Editor data={editorData} onChange={(data) => setEditorData(data)} />
      </div>
      <Button onClick={handlePublish} disabled={!title || !editorData}>
        Publish
      </Button>
    </Container>
  );
}
```

Save everything, make sure the app server is still running, and open [localhost:3000/blog/new](http://localhost:3000/blog/new) in the browser. Write your first blog and hit publish. You will be landing on the home page.

Feeling curious? Check out the `blog.sqlite` file, you will see the blog has been saved, along with a link to the `userId` of its proud author.

## Main Layout

Till now our website is not having any layout, Let's create one.

🛠️ Create a new file as `header.tsx` inside `src/components/custom`, then copy & paste the below code.

```jsx
import { LogoutAction } from "@/app/login/action";
import Link from "next/link";

export default function Header() {
  return (
    <header className="flex flex-row justify-between items-center py-4 px-6 bg-gradient-to-r from-indigo-200 to-emerald-200">
      <div className="flex flex-row gap-4 items-center justify-start">
        <Link href={"/"} className="inline-block font-semibold text-2xl">
          Blogger
        </Link>
        <Link href={"/blog/new"}>Write Blog</Link>
      </div>
      <form action={LogoutAction}>
        <button
          type="submit"
          className="bg-white text-black rounded-2xl py-0.5 px-4 text-sm hover:bg-black hover:text-white transition-all duration-300 ease-out"
        >
          Logout
        </button>
      </form>
    </header>
  );
}
```

🛠️ Open the `src/app/login/action.tsx` file and add the Logout action.

```ts
export async function LogoutAction() {
  const cookieStore = await cookies();
  cookieStore.delete("session");
  redirect("/login");
}
```

🛠️ Create a new file as `sharedLayout.tsx` inside `src/components/custom`, then copy & paste the below code.

```jsx
"use client";

import { usePathname } from "next/navigation";
import React from "react";
import Header from "./header";

export default function SharedLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const pathName = usePathname();
  const nonLayoutRoutes = ["/login", "/register"];
  const shouldRenderLayout = !nonLayoutRoutes.includes(pathName);
  return (
    <main>
      {shouldRenderLayout ? (
        <>
          <Header />
          <main>{children}</main>
        </>
      ) : (
        <main>{children}</main>
      )}
    </main>
  );
}
```

🛠️ Import the SharedLayout component inside `src/app/layout.tsx` file and use it as below.

```jsx
export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body
        className={`${geistSans.variable} ${geistMono.variable} antialiased`}
      >
        <SharedLayout>{children}</SharedLayout>
      </body>
    </html>
  );
}
```

Alright, time to shine! Save all the files, and you will notice the header in action. You can navigate to the blog writing page or even log out like a pro.

Keep in mind, the shared layout wil not show up on the login and register pages, that's not a bug! We excluded them in the `nonLayoutRoutes` array. No freeloading layouts here! 😜

## Display Blogs

Time to bring your blogs to life, let's display that list and make them shine!

🛠️ Open the `src/app/page.tsx` file, then copy & paste the below code.

```tsx
import { Container } from "@/components/custom/container";
import Timestamp from "@/components/custom/timestamp";
import { Blog } from "@/db/models/blog";
import { User } from "@/db/models/user";
import Link from "next/link";

export default async function Home() {
  const blogs = await Blog.findAll({
    include: User,
    order: [["createdAt", "DESC"]],
  });
  return (
    <Container className="flex flex-col gap-4 mt-8">
      {blogs.map((b, i) => (
        <Link
          key={b.id}
          href={`/blog/${b.id}`}
          className="px-8 py-4 border rounded-md shadow"
        >
          <div className="flex flex-row justify-between items-center">
            <p className="text-xl font-medium">{b.title}</p>
            <p className="text-sm font-semibold">
              written by {b?.User?.firstName} {b?.User?.lastName}
            </p>
          </div>
          <Timestamp date={new Date(b?.createdAt || "")} />
        </Link>
      ))}
    </Container>
  );
}
```

🛠️ Create a new file as `timestamp.tsx`  inside `src/components/custom` folder, then copy & paste the below code.

```jsx
"use client";

import React from "react";
import { formatDistanceToNow } from "date-fns";

type Props = {
  date: Date;
};

export default function Timestamp({ date }: Props) {
  const timeAgo = formatDistanceToNow(date, { addSuffix: true });
  return (
    <p className="text-sm text-gray-800">
      {timeAgo.replace("about", "").trim()}
    </p>
  );
}
```

🛠️ Install the date-fns library by running `npm i date-fns` in terminal.

Save your changes, refresh the home page, and there it is, the blog you wrote earlier proudly displayed.

Now, try writing another blog. Notice how it jumps straight to the top? That's because we are sorting them by `createdDate` in descending order, because your freshest thoughts deserve the spotlight.😃

## Dynamic Routing

Since the blogs are dynamic to show the full blog we should have a dynamic route, let's create one.

In Next.js a dynamic route segment can be created by wrapping a folder's name in square brackets like 👉 [folderName]

🛠️ create a new folder as `[blogId]` inside `src/app/blog`.

🛠️ create a `page.tsx` in the new folder, then copy & paste the below code.

```tsx
import { Container } from "@/components/custom/container";
import Timestamp from "@/components/custom/timestamp";
import { Blog } from "@/db/models/blog";
import { User } from "@/db/models/user";
import React from "react";

type Props = {
  params: Promise<{ blogId: string }>;
};

export default async function BlogDetailsPage({ params }: Props) {
  const { blogId } = await params;
  const blog = await Blog.findByPk(blogId, { include: User });

  return (
    <Container className="mt-8">
      <div className="px-8 py-4 border rounded-md shadow">
        <div className="mb-4 border-b pb-4">
          <div className="flex flex-row justify-between items-center">
            <p className="text-xl font-medium">{blog?.title}</p>
            <p className="text-sm font-semibold">
              written by {blog?.User?.firstName} {blog?.User?.lastName}
            </p>
          </div>
          <Timestamp date={new Date(blog?.createdAt || "")} />
        </div>
        <div dangerouslySetInnerHTML={{ __html: blog?.content || "" }} />
      </div>
    </Container>
  );
}
```

Save all your changes and navigate to the home page. Click on any blog, and you will be redirected to its dynamic blog route.

Using the `blogId`, we fetch the details from the database and display the entire blog beautifully.

Well, that's a wrap for today's session! Thanks for following along, and I will see you next time!
