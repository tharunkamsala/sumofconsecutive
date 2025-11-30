# Video Demo Guide

Use this script to record two videos: (1) Core features, (2) Bonus signatures. Show the server logs (Terminal 1) and the CLI interactions (Terminals 2/3).

## Core Features Demo

**Terminal 1 (Server)**
```bash
gleam run -m reddit_clone/rest_main -- 8081
```
Keep this visible to capture requests.

**Terminal 2 (User A: alice)**
```bash
gleam run -m reddit_clone/rest_cli_main -- --port 8081
register alice
login alice
status online
create-sub news
join-sub news
post news keys/alice_private.pem "Hello from Alice!"
feed
post-detail 1        # use actual ID from feed if different
my-posts
metrics              # shows totals + online users
```

**Terminal 3 (User B: bob)**
```bash
gleam run -m reddit_clone/rest_cli_main -- --port 8081
register bob
login bob
status online
join-sub news
feed                      # sees Alice’s post
comment 1 "Hi Alice!"     # use actual post ID from feed
vote-post 1 up
dm alice "Nice post!"
dm-threads
metrics
```

**Wrap-up (either terminal)**
```bash
status offline
metrics   # online_users should drop; totals remain
```

**What to show:** server log output; CLI showing IDs; feeds; comments/votes; DMs (`dm-threads`); detailed `metrics` (uptime, ops, posts, comments, votes, DMs, peak online users).

## Bonus Signatures Demo

**Terminal 1 (Server)**
```bash
gleam run -m reddit_clone/rest_main -- 8081
```

**Terminal 2 (User: alice)**
```bash
gleam run -m reddit_clone/rest_cli_main -- --port 8081
register alice
ls keys/alice_*pem             # show key files
head -n 3 keys/alice_public.pem  # optional peek
login alice
create-sub news
join-sub news
post news keys/alice_private.pem "Signed hello"
feed
post-detail 1                   # use actual ID; signature verified on fetch
curl http://localhost:8081/api/v1/public_keys/alice   # public key lookup
```

**Negative case (wrong key)**
```bash
post news keys/bob_private.pem "Should fail"   # expect signature_rejected/400
```

**Show metrics**
```bash
metrics
```

**What to show:** key generation on register; public key lookup; successful signed post; `post-detail` after re-verification; failed post with wrong key; metrics.

## One-command scripted demo (optional)
If you want an automated run, use:
```bash
PORT=8081 bash test_script.sh
```
It starts the server if the port is free and runs the full flow (register, status, subs, signed posts, comments/replies, votes, DM, feeds, post-detail, metrics).



# Public key lookup
curl http://localhost:8081/api/v1/public_keys/tharun
curl http://localhost:8081/api/v1/public_keys/sravya

# Feed (use the session token printed on login for each user)
curl -H "Authorization: Bearer <tharun_token>" http://localhost:8081/api/v1/feed/tharun
curl -H "Authorization: Bearer <sravya_token>" http://localhost:8081/api/v1/feed/sravya

# Post detail
curl -H "Authorization: Bearer <sravya_token>" http://localhost:8081/api/v1/posts/1

# Metrics
curl -H "Authorization: Bearer <tharun_token>" http://localhost:8081/api/v1/metrics

