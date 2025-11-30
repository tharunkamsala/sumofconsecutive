# Reddit Clone – Bonus: Public Key Signatures

This document explains how the bonus public-key signature features work and how to demo them.

## What’s Implemented
- **Public key on registration:** `register <user>` requires/provides a public key. The CLI auto-generates RSA keys and saves `keys/<user>_public.pem` and `keys/<user>_private.pem`. You can also supply your own public key path.
- **Public key discovery:** `GET /api/v1/public_keys/:username` returns the stored public key.
- **Signed posts:** Posts/reposts must include a signature (hex) generated with the poster’s private key over the payload `<username>|<subreddit>|<content>`. The server verifies at creation.
- **Verification on download:** Every fetch (feed, post detail, user posts) re-verifies the stored signature. If invalid, the request fails with `signature_invalid`. Responses include `signature_valid` but do not expose the raw signature.
- **Login auth:** `login <user>` signs a fixed challenge with the private key; server verifies against the stored public key.

## How to Demo (CLI)
Prereq: Start server `gleam run -m reddit_clone/rest_main -- 8081`, start CLI `gleam run -m reddit_clone/rest_cli_main -- --port 8081`.

1) **Register (keys generated)**
   ```
   register alice
   ls keys/alice_*pem
   head -n 3 keys/alice_public.pem   # optional peek
   ```
2) **Login (signature-based)**
   ```
   login alice
   ```
   (Fails if the private key is missing/wrong.)
3) **Create & join subreddit**
   ```
   create-sub news
   join-sub news
   ```
4) **Post with signature**
   ```
   post news keys/alice_private.pem "Signed hello"
   feed           # shows the post; signature was verified on create and on fetch
   post-detail 1  # re-verifies on download; shows signature_valid true
   ```
5) **Show public key lookup**
   ```
   curl http://localhost:8081/api/v1/public_keys/alice
   ```
6) **Show rejection on bad signature** (point to a wrong key)
   ```
   post news keys/tharun_private.pem "Should fail"   # expect signature_rejected
   ```

## Key Endpoints
- Register: `POST /api/v1/accounts` (username, public_key)
- Public key lookup: `GET /api/v1/public_keys/:username`
- Create post: `POST /api/v1/posts` (signature required)
- Repost: `POST /api/v1/posts/:id/repost` (signature required)
- Post detail: `GET /api/v1/posts/:id` (re-verifies signature)
- Feed: `GET /api/v1/feed/:user` (re-verifies signature on posts)
- User posts: `GET /api/v1/accounts/:user/posts`
- Login: `POST /api/v1/auth/login` (signature over `<user>|auth|login`)

## Notes
- Signatures are hex; verification uses the stored public key.
- Responses hide the raw signature but include `signature_valid`.
- Data is in-memory per server run; IDs reset on fresh start.
