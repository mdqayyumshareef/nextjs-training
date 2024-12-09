# Welcome to the session

Alright, let's pick up right where we left off last time and dive into making some real progress on our blog project.

## JWT Setup

As part of the login process, We will issue a session token to make sure the protected routes are only accessible to authenticated users. It’s basically our app’s way of checking IDs at the door.

🛠️ Install the jose library by running `npm i jose` in terminal.

🛠️ Create a new file as `encryption.ts` inside `src/lib` folder, then copy & paste the below code.

```ts
import { SignJWT, jwtVerify } from "jose";

type tokenPayload = {
  userDetails: {
    id: string;
    firstName: string | null;
    lastName: string | null;
    email: string;
  };
};

const secret = "let_token = null; // noOneWillGuessThisOne!;)";
const key = new TextEncoder().encode(secret);
const alg = "HS256";

export async function encrypt(payload: tokenPayload) {
  return await new SignJWT(payload)
    .setProtectedHeader({ alg })
    .setIssuedAt()
    .setExpirationTime("1 day")
    .sign(key);
}

export async function decrypt(input: string) {
  try {
    const { payload } = await jwtVerify<tokenPayload>(input, key, {
      algorithms: [alg],
    });
    return payload;
  } catch (e) {
    console.error(e);
  }
}
```

## Login Server Action

🛠️ Create a new file as `action.ts` inside login folder, then copy & paste the below code.

```jsx
"use server";

import { User } from "@/db/models/user";
import { encrypt } from "@/lib/encryption";
import bcrypt from "bcrypt";
import { cookies } from "next/headers";
import { redirect } from "next/navigation";

export async function LoginAction(data: FormData) {
  try {
    const loginEmail = data.get("email")?.toString();
    const loginPassword = data.get("password")?.toString();
    if (!loginEmail || !loginPassword) {
      throw new Error("Missing credentials");
    }

    const user = await User.findOne({ where: { email: loginEmail } });
    if (!user) {
      throw new Error("Invalid credentials");
    }

    const match = await bcrypt.compare(loginPassword, user?.password);
    if (!match) {
      throw new Error("Invalid credentials");
    }

    const token = await encrypt({
      userDetails: {
        id: user.id,
        firstName: user.firstName,
        lastName: user.lastName,
        email: user.email,
      },
    });

    const cookieStore = await cookies();
    const cookieExpireTime = 10 * 60 * 1000; // 10 minutes
    cookieStore.set("session", token, {
      expires: new Date(Date.now() + cookieExpireTime),
      httpOnly: true,
      secure: true,
    });
    redirect("/");
  } catch (e) {
    throw e;
  }
}
```

Seems like Typescript is upset, it's shouting *Property 'password' does not exist on type 'Model<any, any>'*

Don't worry Typescript is just being overprotective. Let's calm it down and fix it

### Add Types to User Model

🛠️ Open the `user.ts` file, then copy & paste the below code.

```jsx
import { DataTypes, Model, Optional } from "sequelize";
import { sequelize } from "..";

interface UserAttributes {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  password: string;
}

type UserCreationAttributes = Optional<UserAttributes, "id">;

interface UserInstance
  extends Model<UserAttributes, UserCreationAttributes>,
    UserAttributes {
  createdAt?: Date;
  updatedAt?: Date;
}

export const User = sequelize.define<UserInstance>("Users", {
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

Now, if you have a look at the LoginAction we are no longer getting the Typescript error.

### Connect the Login Server Action

🛠️ Open the login page, and pass the LoginAction in the form action as below.

```html
  <form action={LoginAction}>
    // nested code
  </form>
```

Save your changes and head to the browser. On the login page, punch in your email and password, then hit submit like a pro. 👨‍💻

Behind the scenes, after verifying the user credentials, an encrypted token has been created and safely stashed in a cookie.

Curious? Open your browser's dev tools, go to the Application tab, and check the Cookies section under Storage.

You should see the session cookie there, proudly holding your token. If you don't, double-check your code or bribe your browser with more cookies. 😜

## Error Handling in Login Page

🛠️ Create a new file as `error.tsx` inside login folder. then copy & paste the below code.

```jsx
"use client"; // Error boundaries must be Client Components

import { Container } from "@/components/custom/container";
import { Button } from "@/components/ui/button";
import { useEffect } from "react";

type Props = {
  error: Error;
  reset: () => void;
};

export default function Error({ error, reset }: Props) {
  useEffect(() => {
    // In client projects you may want to log the error to an error reporting service.
    console.error(error);
  }, [error]);

  return (
    <Container>
      <p className="mb-4">Something went wrong, {error.message}</p>
      <Button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </Button>
    </Container>
  );
}
```

Let's put the error page to the test! Manually throw an error from the `LoginAction` and let the error page show off its troubleshooting skills.

🛠️ In the LoginAction comment the entire function body, copy & paste the below code.

```jsx
throw new Error("I love debugging!");
```

Save your changes, head to the login page, and submit the form.

You will be greeted by the error page (don't panic, it's intentional). Now, click the `Try again` button, and you are back to the login form, ready for another go! 😅

Before moving on, Make sure you remove the exception and bring the LoginAction body back to life by uncommenting it. Let's keep things on track!

## Blog Writing Page

Let’s dive into the blog writing page. we will need a text editor for crafting all that amazing blog content

🛠️ Install CKEditor by running `npm i @ckeditor/ckeditor5-react @ckeditor/ckeditor5-build-classic`

🛠️ Create a new file as `editor.tsx` at `src/components/custom` then copy & paste below code.

```jsx
"use client";

import { CKEditor } from "@ckeditor/ckeditor5-react";
import ClassicEditor from "@ckeditor/ckeditor5-build-classic";

type props = {
  data: string;
  onChange: (data: string) => void;
};

export const Editor = ({ data, onChange }: props) => {
  return (
    <CKEditor
      editor={ClassicEditor}
      data={data}
      onChange={(event, editor) => {
        const data = editor.getData();
        onChange(data);
      }}
      config={{
        licenseKey: "GPL",
        placeholder: "Type your content here...",
      }}
    />
  );
};
```

🛠️ In the `app` directory create a folder as `blog` then create another nested folder name it as `new`. Create a file as `page.tsx`, copy & paste the below code.

```jsx
"use client";

import { Container } from "@/components/custom/container";
import { Editor } from "@/components/custom/editor";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { useState } from "react";

export default function Blog() {
  const [editorData, setEditorData] = useState("");

  return (
    <Container>
      <h1 className="mb-8">From Thoughts to Words: Blog Without Limits</h1>
      <Label htmlFor="title" className="inline-block mb-2">
        Title
      </Label>
      <Input id="title" />
      <div className="my-4">
        <Editor data={editorData} onChange={(data) => setEditorData(data)} />
      </div>
      <Button>Publish</Button>
    </Container>
  );
}
```

🛠️ Add default heading styles, let's open the `global.css` file, copy & pasting below code.

```css
@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }

  h1 {
    @apply text-3xl font-semibold;
  }
  h2 {
    @apply text-2xl font-semibold;
  }
  h3 {
    @apply text-xl font-semibold;
  }
  h4 {
    @apply text-lg font-semibold;
  }
  h5 {
    @apply text-base font-semibold;
  }
  h6 {
    @apply text-sm font-semibold;
  }
}
```

Let's head over to the browser and manually navigate to the blog writing page by entering <http://localhost:3000/blog/new> in the address bar. Looks good, doesn’t it?

## Session Management

Notice something? Our blog writing page isn’t protected, it’s like leaving the front door open. That’s not ideal. We want users to log in first before they unleash their inner blogger.

So, let’s add a session token check. If there is no token, we will politely redirect users to the login page.

🛠️ Create a new file as `session.tsx` inside `src/lib` then copy & paste the below code.

```ts
import { cookies } from "next/headers";
import { decrypt } from "./encryption";
import { NextRequest, NextResponse } from "next/server";

export async function getSession(request?: NextRequest) {
  let token;
  if (request) {
    token = request.cookies.get("session")?.value;
  } else {
    const cookieStore = await cookies();
    token = cookieStore.get("session")?.value;
  }
  if (!token) return null;
  return await decrypt(token);
}

export async function updateSession(request: NextRequest) {
  const token = request.cookies.get("session")?.value;
  if (!token) return;

  const res = NextResponse.next();
  const cookieExpireTime = 10 * 60 * 1000; // 10 minutes
  res.cookies.set({
    name: "session",
    value: token,
    expires: new Date(Date.now() + cookieExpireTime),
    httpOnly: true,
    secure: true,
  });
  return res;
}
```

🛠️ Create a new file as `middleware.ts` inside `src` folder, then copy & paste the below code.

```ts
import { getSession, updateSession } from "@/lib/session";
import { cookies } from "next/headers";
import { NextResponse, type NextRequest } from "next/server";

export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */
    "/((?!api|_next/static|_next/image|favicon.ico|.*\\.png$).*)",
  ],
};

const shouldRedirect = (url: string) => {
  const unprotectedRoutes = ["/login", "/register"];
  let flag = true;
  for (let i = 0; i < unprotectedRoutes.length; i++) {
    if (new URL(url).pathname === unprotectedRoutes[i]) {
      flag = false;
      break;
    }
  }
  return flag;
};

export async function middleware(request: NextRequest) {
  const tokenPayload = await getSession(request);
  if (tokenPayload) {
    return updateSession(request);
  }

  if (!tokenPayload && shouldRedirect(request.url)) {
    const cookieStore = await cookies();
    cookieStore.delete("session");
    return NextResponse.redirect(new URL("/login", request.url));
  }
}
```

Alright, save all the files. Time to check out our work!

Clear your cookies, then try accessing the blog writing page. Don’t be shocked when you get denied, it’s protected now, only rolling out the red carpet for authenticated users. Unauthenticated? Sorry, no VIP access for you!

That's a wrap for today! You are now officially one step closer to being the person who says "It’s simple, just use React and Next.js!" and actually means it. 😁

See you next time!
