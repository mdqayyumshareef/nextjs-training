# Welcome to the session

Alright, let's continue where we left off in the last session

## Loading UI

🛠️ Create a new file as `spinner.tsx` inside `src/components/custom` then copy & paste the below code.

```tsx
import React from "react";

export function Spinner() {
  return (
    <div role="status" className="flex justify-center items-center">
      <svg
        aria-hidden="true"
        className="inline w-10 h-10 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600"
        viewBox="0 0 100 101"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
          fill="currentColor"
        />
        <path
          d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
          fill="currentFill"
        />
      </svg>
    </div>
  );
}
```

🛠️ Create a new file as `loading.tsx` inside `src/app/blog/[blogId]` then copy & paste the below code.

```tsx
import { Container } from "@/components/custom/container";
import { Spinner } from "@/components/custom/spinner";
import React from "react";

export default function Loading() {
  return (
    <Container>
      <Spinner />
    </Container>
  );
}
```

Loading file can create instant loading states built on Suspense. By default, that is a Server Component, but can also be used as a Client Component through the `use client` directive.

Save the changes and navigate to the blog details, you will be seeing a glimpse of spinner just before the content gets displayed.

## Public Blogs Page

Not everyone wants to sign in just to read a blog. Let’s fix that by making our content available to all visitors. Ready? Let’s code it!

🛠️ Open the `middleware.ts` file and update the below code.

```ts
const shouldRedirect = (url: string) => {
  const unprotectedRoutes = ["/", "/blog", "/login", "/register"];
  let flag = true;
  const urlPath = new URL(url).pathname;
  for (let i = 0; i < unprotectedRoutes.length; i++) {
    if (urlPath === unprotectedRoutes[i]) {
      flag = false;
      break;
    }
  }
  if (urlPath.startsWith("/blog") && urlPath !== "/blog/new") {
    flag = false;
  }
  return flag;
};
```

Saving the changes, now visitors should be able to read blogs without logging in.

## Fixing the Header

Notice how the header stays the same no matter what? Let’s level up its functionality and make it dynamic based on the user’s login state.

🛠️ Open the `src/components/custom/header.tsx` file, copy & paste the below code.

```tsx
import { LogoutAction } from "@/app/login/action";
import { getSession } from "@/lib/session";
import Link from "next/link";
import { Button } from "../ui/button";

export default async function Header() {
  const token = await getSession();
  const userId = token?.userDetails?.id;
  const userName =
    `${token?.userDetails?.firstName} ${token?.userDetails?.lastName}`.trim();

  return (
    <header className="flex flex-row justify-between items-center py-4 px-6 bg-gradient-to-r from-indigo-200 to-emerald-200">
      <div className="flex flex-row gap-4 items-center justify-start">
        <Link href={"/"} className="inline-block font-semibold text-2xl">
          Blogger
        </Link>
        <Link href={"/blog/new"}>Write Blog</Link>
      </div>
      <div>
        {userId ? (
          <div className="flex items-center justify-start">
            <span className=" font-semibold">Welcome {userName}</span>
            <Button variant="link" asChild>
              <Link href={`/author/${userId}`}>My Blogs</Link>
            </Button>
            <form action={LogoutAction}>
              <Button type="submit">Logout</Button>
            </form>
          </div>
        ) : (
          <Button asChild>
            <Link href="/login">Login</Link>
          </Button>
        )}
      </div>
    </header>
  );
}
```

🛠️ Open the `src/components/custom/sharedLayout.tsx` file, copy & paste the below code.

```tsx
"use client";

import { usePathname } from "next/navigation";
import React from "react";

export default function SharedLayout({
  header,
  children,
}: {
  header: React.ReactNode;
  children: React.ReactNode;
}) {
  const pathName = usePathname();
  const nonLayoutRoutes = ["/login", "/register"];
  const shouldRenderLayout = !nonLayoutRoutes.includes(pathName);
  return (
    <main>
      {shouldRenderLayout ? (
        <>
          {header}
          <main>{children}</main>
        </>
      ) : (
        <main>{children}</main>
      )}
    </main>
  );
}
```

🛠️ Update the `src/app/layout.tsx` file as below.

```tsx
<SharedLayout header={<Header />}>{children}</SharedLayout>
```

Save your files and check out the header. You will notice it changes dynamically depending on whether the user is logged in or logged out.

## Author's Blog Edit & Delete

🛠️ Create a new folder as `author/[authorId]` inside `src/app` directory.

🛠️ Create a new file as `page.tsx` inside `src/app/author/[authorId]` folder, then copy & paste the below code.

```tsx
import { Container } from "@/components/custom/container";
import Timestamp from "@/components/custom/timestamp";
import { Button } from "@/components/ui/button";
import { Blog } from "@/db/models/blog";
import { FilePenLine, SquareChartGantt, Trash2 } from "lucide-react";
import Link from "next/link";
import { DeleteBlogAction } from "./action";

type Props = {
  params: Promise<{ authorId: string }>;
};

export default async function AuthorBlogsPage({ params }: Props) {
  const { authorId } = await params;

  const blogs = await Blog.findAll({
    where: {
      userId: authorId,
    },
  });
  return (
    <Container className="flex flex-col gap-4 mt-8">
      {blogs.map((b, i) => (
        <div className="px-8 py-4 border rounded-md shadow" key={b.id}>
          <div className="flex flex-row justify-between items-center">
            <p className="text-xl font-medium">{b.title}</p>
            <div className="flex flex-row gap-2">
              <Button type="button" asChild variant="outline">
                <Link href={`/blog/${b.id}`}>
                  <SquareChartGantt /> View
                </Link>
              </Button>
              <Button type="button" asChild variant="outline">
                <Link
                  href={{
                    pathname: "/blog/edit",
                    query: { blogId: b.id },
                  }}
                >
                  <FilePenLine /> Edit
                </Link>
              </Button>
              <form action={DeleteBlogAction}>
                <input hidden type="text" name="blogId" defaultValue={b.id} />
                <input
                  hidden
                  type="text"
                  name="authorId"
                  defaultValue={authorId}
                />
                <Button type="submit" variant="outline">
                  <Trash2 /> Delete
                </Button>
              </form>
            </div>
          </div>
          <Timestamp date={new Date(b?.createdAt || "")} />
        </div>
      ))}
    </Container>
  );
}
```

🛠️ Create a new file as `action.ts` inside `src/app/author/[authorId]` folder, then copy & paste the below code.

```ts
"use server";

import { Blog } from "@/db/models/blog";
import { redirect } from "next/navigation";

export async function DeleteBlogAction(formData: FormData) {
  const blogId = formData.get("blogId")?.toString();
  const authorId = formData.get("authorId")?.toString();
  await Blog.destroy({ where: { id: blogId } });
  redirect(`/author/${authorId}`);
}
```

🛠️ Create a new folder as `edit` inside `src/app/blog` directory.

🛠️ Create a new file as `page.tsx` inside `src/app/blog/edit` folder, then copy & paste the below code.

```tsx
import { Container } from "@/components/custom/container";
import { Blog } from "@/db/models/blog";
import BlogEdit from "./blog-edit";

type Props = {
  searchParams: Promise<{ [key: string]: string }>;
};

export default async function BlogEditPage({ searchParams }: Props) {
  const query = await searchParams;
  const blogId = query?.blogId;
  const blog = await Blog.findByPk(blogId);

  return (
    <Container>
      <h1 className="mb-8">From Thoughts to Words: Blog Without Limits</h1>
      <BlogEdit
        blogId={blogId}
        blogTitle={blog?.title || ""}
        blogContent={blog?.content || ""}
      />
    </Container>
  );
}
```

🛠️ Create a new file as `blog-edit.tsx` inside `src/app/blog/edit` folder, then copy & paste the below code.

```tsx
"use client";

import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import dynamic from "next/dynamic";
import { useState } from "react";
import { UpdateBlogAction } from "./action";

const Editor = dynamic(
  () => import("@/components/custom/editor").then((x) => x.Editor),
  {
    ssr: false,
    loading: () => <p>Loading...</p>,
  }
);

type Props = {
  blogId: string;
  blogTitle: string;
  blogContent: string;
};

export default function BlogEdit({
  blogId,
  blogTitle = "",
  blogContent = "",
}: Props) {
  const [title, setTitle] = useState(blogTitle);
  const [editorData, setEditorData] = useState(blogContent);

  const handlePublish = () => {
    UpdateBlogAction({
      blogId,
      title,
      content: editorData,
    });
  };

  return (
    <>
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
    </>
  );
}
```

🛠️ Create a new file as `action.tsx` inside `src/app/blog/edit` folder, then copy & paste the below code.

```ts
"use server";

import { Blog } from "@/db/models/blog";
import { redirect } from "next/navigation";

type Params = {
  blogId: string;
  title: string;
  content: string;
};

export async function UpdateBlogAction({ blogId, title, content }: Params) {
  await Blog.update(
    { title, content },
    {
      where: {
        id: blogId,
      },
    }
  );
  redirect(`/blog/${blogId}`);
}
```

🛠️ Replace the `shouldRedirect` function body in the `middleware.ts` with the below code.

```ts
const shouldRedirect = (url: string) => {
  const unprotectedRoutes = ["/", "/blog", "/login", "/register"];
  let flag = true;
  const urlPath = new URL(url).pathname;
  for (let i = 0; i < unprotectedRoutes.length; i++) {
    if (urlPath === unprotectedRoutes[i]) {
      flag = false;
      break;
    }
  }
  if (
    urlPath.startsWith("/blog") &&
    !["/blog/new", "/blog/edit"].includes(urlPath)
  ) {
    flag = false;
  }
  return flag;
};
```

Save all your files, and explore the ‘My Blogs’ link in the header—it gives you access to all blogs created by the logged-in user.

Take a closer look at the View, Edit, and Delete buttons in the blog list:

* The Edit button takes you to the blog edit page with the blog content preloaded in the text editor. Make some changes, save them, and watch as the updates reflect in the database.

* The Delete button does exactly what you’d expect—removes the blog from the database with ruthless efficiency.

Go ahead and give it a try!

Well, That's all for today, we will catch up later. See you!
