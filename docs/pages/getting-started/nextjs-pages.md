---
title: "Getting started in Next.js Pages router"
---

# Getting started in Next.js Pages router

## Installation

Install Lucia using your package manager of your choice. While not strictly necessary, we recommend installing [Oslo](https://oslo.js.org), which Lucia is built on, for various auth utilities (which a lot of the guides use).

```
npm install lucia oslo
```

## Initialize Lucia

Import `Lucia` and initialize it with your adapter. Refer to the [Database](/database) page to learn how to set up your database and initialize the adapter. Make sure you configure the `sessionCookie` option and register your `Lucia` instance type.

```ts
// src/auth.ts
import { Lucia } from "lucia";

const adapter = new BetterSQLite3Adapter(db); // your adapter

export const lucia = new Lucia(adapter, {
	sessionCookie: {
		attributes: {
			// set to `true` when using HTTPS
			secure: process.env.NODE_ENV === "production"
		}
	}
});

// IMPORTANT!
declare module "lucia" {
	interface Register {
		Lucia: typeof lucia;
	}
}
```

## Polyfill

If you're using Node.js 18 or below, you'll need to polyfill the Web Crypto API. This is not required in Node.js 20, CouldFlare Workers, Deno, Bun, and Vercel Edge Functions. This can be done either by importing `webcrypto`, or by enabling an experimental flag.

```ts
import { webcrypto } from "node:crypto";

globalThis.crypto = webcrypto as Crypto;
```

```
node --experimental-web-crypto index.js
```

## Set up middleware

If you're planning to use cookies, you must implement CSRF protection.

```ts
// middleware.ts
import { verifyRequestOrigin } from "lucia";
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export async function middleware(request: NextRequest): Promise<NextResponse> {
	if (request.method === "GET") {
		return NextResponse.next();
	}
	const originHeader = request.headers.get("Origin");
	const hostHeader = request.headers.get("Host");
	if (!originHeader || !hostHeader || !verifyRequestOrigin(originHeader, [hostHeader])) {
		return new NextResponse(null, {
			status: 403
		});
	}
	return NextResponse.next();
}
```

## Next steps

You can learn all the concepts and APIs by reading the [Basics section](/basics/sessions) in the docs. If you prefer writing code immediately, check out the [Tutorials](/tutorials) page or the [examples repository](https://github.com/lucia-auth/examples/tree/v3).

If you have any questions, [join our Discord server](https://discord.com/invite/PwrK3kpVR3)!
