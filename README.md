# Redis Docker Server

Public Redis on host port **9898** with **multiple usernames and passwords** (Redis ACL). Push to GitHub, pull on your VPS, run with Docker Compose.

Domain: **redis.dddemo.net** → connect from any server with a Redis URL.

## 1. Push to GitHub

```bash
git init
git add .
git commit -m "Add Redis Docker Compose setup"
git remote add origin YOUR_GITHUB_REPO_URL
git push -u origin main
```

Do **not** commit `users.acl` — it is gitignored. Only `users.acl.example` is in the repo.

## 2. Point the domain (DNS)

In your DNS panel for `dddemo.net`, create an **A record**:

| Type | Name / Host | Value       | TTL |
|------|-------------|-------------|-----|
| A    | redis       | YOUR_VPS_IP | 300 |

Open port **9898** on the VPS:

```bash
sudo ufw allow 9898/tcp
sudo ufw reload
```

## 3. On the VPS

```bash
git clone YOUR_GITHUB_REPO_URL
cd RedisDocker   # or your repo folder name

cp users.acl.example users.acl
nano users.acl   # set strong passwords for each user

docker compose up -d
```

Later updates:

```bash
git pull
docker compose up -d
```

## 4. Connect with username + password (URL)

```
redis://USERNAME:PASSWORD@redis.dddemo.net:9898
```

Examples from `users.acl.example`:

```
redis://default:CHANGE_DEFAULT_PASSWORD@redis.dddemo.net:9898
redis://app1:CHANGE_APP1_PASSWORD@redis.dddemo.net:9898
redis://app2:CHANGE_APP2_PASSWORD@redis.dddemo.net:9898
```

### App examples

**redis-cli**

```bash
redis-cli -h redis.dddemo.net -p 9898 --user app1 -a YOUR_PASSWORD ping
```

**Node**

```js
const url = "redis://app1:YOUR_PASSWORD@redis.dddemo.net:9898";
```

**Laravel `.env`**

```env
REDIS_HOST=redis.dddemo.net
REDIS_PORT=9898
REDIS_USERNAME=app1
REDIS_PASSWORD=YOUR_PASSWORD
```

**Python**

```python
r = redis.from_url("redis://app1:YOUR_PASSWORD@redis.dddemo.net:9898")
```

## 5. Create more users on the server

While Redis is running, create a new user (use an existing admin/default password):

```bash
docker exec -it redis-server redis-cli --user default -a 'YOUR_DEFAULT_PASSWORD' \
  ACL SETUSER myapp on '>MyStrongPass123!' '~*' '&*' '+@all'

docker exec -it redis-server redis-cli --user default -a 'YOUR_DEFAULT_PASSWORD' ACL SAVE
```

That writes the new user into `users.acl` on the host so it survives restarts.

List users:

```bash
docker exec -it redis-server redis-cli --user default -a 'YOUR_DEFAULT_PASSWORD' ACL LIST
```

Delete a user:

```bash
docker exec -it redis-server redis-cli --user default -a 'YOUR_DEFAULT_PASSWORD' ACL DELUSER myapp
docker exec -it redis-server redis-cli --user default -a 'YOUR_DEFAULT_PASSWORD' ACL SAVE
```

Or edit `users.acl` on the host and restart:

```bash
nano users.acl
docker compose restart
```

### Read-only user example

```bash
docker exec -it redis-server redis-cli --user default -a 'YOUR_DEFAULT_PASSWORD' \
  ACL SETUSER readonly on '>ReadOnlyPass!' '~*' '&*' '+@read'

docker exec -it redis-server redis-cli --user default -a 'YOUR_DEFAULT_PASSWORD' ACL SAVE
```

## Useful commands

```bash
docker compose ps
docker compose logs -f redis
docker compose restart
docker compose down
```

Redis keeps running after reboot (`restart: unless-stopped`). Data is stored in the `redis_data` Docker volume. Users live in `users.acl`.
# redis
