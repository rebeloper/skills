# Security checklists

Read a section only when the diff touches its area (tier 2: inside the C bug hunt; tier 3+: in the H security pass). Each prompt is a question about the diff, not about the whole codebase. Every security finding must pass the **security gate** in `review-tiers.md`.

## iOS, mobile, and local IPC

- Do new deep links or universal links that change state, import data, or finish a login require a current session and a one-time, expiring binding?
- Do OAuth, magic-link, or invite callbacks bind to the session, account, and pending action that started them?
- Can a `WKWebView` script bridge (`WKScriptMessageHandler`) be reached by non-bundled origins after a navigation, redirect, or iframe? Are bridge methods allowlisted, and are their arguments validated?
- Did the diff add file or universal access to a web view, or a custom URL scheme handler?
- Do Keychain items use the right accessibility class (and `ThisDeviceOnly` where needed)? Did any token move to `UserDefaults`, files, logs, the pasteboard, or notification previews?
- On logout or account switch, are caches, Core Data or SwiftData stores, widgets, background tasks, and web view storage cleared?
- Do notification actions, widgets, App Intents, and share extensions perform exactly the action the user saw, for the current account?
- Are entitlement, `Info.plist`, and App Transport Security exception changes visible in the diff and justified?

## Client-side web

- Do new `innerHTML`, `dangerouslySetInnerHTML`, `v-html`, or `eval` sinks receive data an attacker can reach?
- Do `postMessage` handlers check the exact origin and source? Is sensitive data ever posted to `*`?
- Are redirect or navigation targets taken from the URL, query, or storage restricted by scheme and host?
- Does a deep merge with user-controlled keys allow prototype pollution, and is there a gadget that makes it matter?
- Do service-worker or cache keys include the user's identity, and are they cleared on logout?
- Does credentialed CORS reflect an unvalidated `Origin`, or allow `null`?
- Can a new state-changing screen be framed by another site?

## Auth and sessions

- Does each new route check authorization for this specific resource, not only that the user is logged in?
- Can a body field override the user, tenant, or role?
- JWT: is the signature verified, the algorithm pinned, and `aud`, `iss`, and `exp` checked? Is there any decode-without-verify path?
- Is the session rotated on login and privilege change, and invalidated on logout and password change?
- Are reset, recovery, and account-linking tokens one-time, expiring, and bound to the user?
- Does MFA or step-up bind to this session and action, and do alternate routes enforce it too?
- Do new cookie-authenticated mutations have CSRF protection?
- Are API keys limited to their scope, and kept out of client bundles, URLs, and logs?

## Data isolation and lifecycle

- Are new queries, cache keys, object paths, and unique constraints scoped by tenant or owner?
- Do raw, unscoped, or service-client queries apply the same filters as the normal path?
- Do exports and bulk endpoints check authorization per item?
- Do imports and restores re-check authorization instead of trusting the archive?
- Does every new read path include the soft-delete filter?
- Do migrations backfill tenant and access fields, and is rollback safe?
- Do signed URLs bind the operation, object, expiry, and tenant?
- Do new log or analytics fields stay free of private content?

## Server input and backend

- Can a user-supplied URL make the server fetch internal addresses (SSRF), including through redirects and DNS rebinding?
- Do user-supplied file names or paths reach the filesystem without normalizing them and checking they stay inside the allowed folder?
- Do uploads check type, size, and content, and are they stored outside the web root with random names?
- Do new SQL, shell, template, or LDAP calls build strings from user input instead of using parameters?
- Do XML parsers or deserializers accept external entities, or arbitrary types from untrusted input?
- Do new endpoints rate-limit expensive or enumerable operations (login, reset, search, export)?

## Supply chain and CI

- Are new or upgraded dependencies pinned, with the lockfile updated? Watch for registry or namespace confusion and new install scripts.
- Are CI actions and container images pinned by commit SHA or digest, not by tag?
- Does a `pull_request_target`, `workflow_run`, or comment trigger run fork code with secrets or a write token?
- Do branch names, PR titles, or issue fields flow into `run:` shell steps or `${{ }}` expressions?
- Did `permissions:` widen, or did new secrets reach jobs that run untrusted code?
- Can a low-trust job write a cache or artifact that a release job later uses?
- Do the build context and ignore rules leak `.env` files, keys, or fixtures into the artifact?
- Are downloaded tools checked against a hash from an independent source?

## LLM and agent code

- Does retrieved or user-supplied content reach tool arguments or privileged actions without a confirmation bound to that exact action?
- Can model output reach HTML, SQL, or shell sinks?
- Can context or memory leak between users?
