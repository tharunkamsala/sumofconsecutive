# Bonus Demo Commands (public key signatures)

Start server:
```
gleam run -m reddit_clone/rest_main -- 8081
```

Open CLI:
```
gleam run -m reddit_clone/rest_cli_main -- --port 8081
```

Commands to show all bonus requirements:
```
# 1) Register (generates keys)
register alice
ls keys/alice_*pem

# 2) Login (signature-based)
login alice

# 3) Create & join subreddit
create-sub news
join-sub news

# 4) Post with signature (verified on create)
post news keys/alice_private.pem "Signed hello"

# 5) Verify on download (signature checked on fetch)
feed                 # response includes signature_valid for each post
post-detail 1        # use actual ID if different; shows signature_valid true

# 6) Public key lookup (via REST, run in a normal terminal)
curl http://localhost:8081/api/v1/public_keys/alice

# 7) Rejection on bad signature (use wrong key)
post news keys/tharun_private.pem "Should fail"

# 8) (Optional) Post detail via REST to see signature_valid
curl http://localhost:8081/api/v1/posts/1    # replace 1 with actual post ID
```

Key endpoints (for reference):
- Register: `POST /api/v1/accounts`
- Public key lookup: `GET /api/v1/public_keys/:username`
- Create post: `POST /api/v1/posts`
- Post detail: `GET /api/v1/posts/:id`
- Feed: `GET /api/v1/feed/:user`
