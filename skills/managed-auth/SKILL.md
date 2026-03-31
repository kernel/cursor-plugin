---
name: managed-auth
description: manage authentication for AI agents — log in and stay logged in across websites. use when setting up login credentials, authentication flows, or persistent authenticated sessions with kernel.
---

# managed auth

managed auth is a standardized way for agents to log in and stay logged in across the internet. kernel securely stores credentials, re-authenticates as needed, and manages authenticated browser sessions.

## how it works

1. create an auth connection for a domain (e.g., github.com, gmail.com)
2. complete a one-time login via a hosted UI or programmatically
3. kernel saves the authenticated session
4. future browsers start already logged in
5. kernel automatically re-authenticates when sessions expire

## quick start

### via CLI

```bash
# create a connection for github
kernel auth connections create --domain github.com --label my-github

# start a login flow
kernel auth connections login <connection-id>
# opens a browser — complete the login manually

# create a browser using the auth connection
kernel browsers create --auth-connection-id <connection-id>
# browser starts already logged in to github
```

### via MCP

1. use `setup_profile` to create a profile with a live login session
2. use `create_browser` with `profile_name` to reuse the session

## supported services

managed auth works with any website. built-in support for:
- gmail / google workspace
- github
- microsoft outlook / office 365
- any custom domain

## connection statuses
- **ACTIVE** — ready to use, credentials valid
- **NEEDS_AUTH** — login expired, needs re-authentication
- **PENDING** — login flow in progress

## tips
- connections persist across browser sessions — set up once, use forever
- kernel handles session refresh and cookie rotation automatically
- use profiles as an alternative for simpler cookie-based auth
- for MFA/2FA, use the hosted UI flow (opens a live browser for manual verification)
