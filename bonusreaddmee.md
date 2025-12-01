# Bonus Demo Commands (public key signatures)

Start server:
```
gleam run -m reddit_clone/rest_main -- 8081
```

Open CLI:
```
gleam run -m reddit_clone/rest_cli_main -- --port 8081
```

Step-by-step commands to show all bonus requirements:
```
# Start server (one terminal)
gleam run -m reddit_clone/rest_main -- 8081

# Open CLI (another terminal)
gleam run -m reddit_clone/rest_cli_main -- --port 8081

# 1) User provides a public key on registration (auto-generates RSA-2048)
register alice
ls keys/alice_*pem
cat keys/alice_public.pem    # optional: show public key

# 2) Login (signature-based) and create/join a subreddit
login alice
create-sub news
join-sub news

# 3) Post with signature (computed at posting)
post news keys/alice_private.pem "Signed hello"

# 4) Signature checked on download
feed                 # shows posts and signature_valid
post-detail 1        # use actual ID; shows signature_valid true

# 5) Any user can fetch another user's public key (run in normal terminal)
curl http://localhost:8081/api/v1/public_keys/alice

# 6) Rejection on bad signature (wrong key)
post news keys/tharun_private.pem "Should fail"

# 7) (Optional) Post detail via REST to see signature_valid
curl http://localhost:8081/api/v1/posts/1    # replace 1 with actual post ID
```

Key endpoints (for reference):
- Register: `POST /api/v1/accounts`
- Public key lookup: `GET /api/v1/public_keys/:username`
- Create post: `POST /api/v1/posts`
- Post detail: `GET /api/v1/posts/:id`
- Feed: `GET /api/v1/feed/:user`
