
# Idea
- A heat map showing all users on this website, according to their last known gps location.


### **Roles**

* **user** → normal dev
* **admin** → same as superuser

  * Sign up only with **secret key** (configurable in `.env`)
  * Can promote/demote other users to/from admin
  * Can add non-registered developers to leaderboards
  * Can view count of all admins in the system

---

### **Auth**

* Supabase Auth (GitHub OAuth for users, Email/Password for admins)
* Admin signup → POST `/api/admin/signup?key=SECRET_KEY`
* Auth stored in secure HttpOnly cookies
* Middleware:

  * Protect `/admin` routes so only `role=admin` can access

---

### **Pages**

1. **Global Leaderboard**
2. **Country Leaderboard**
3. **University Leaderboard**
4. **Profile Page** (public for all users)
5. **Admin Dashboard**

   * View all users
   * Add/edit/remove users
   * Ban/suspend
   * Approve/reject blogs
   * Assign/remove admin role
   * Add users without signup (via GitHub handle fetch)
   * See total number of admins

---

### **Metrics Stored in DB**

* GitHub stars
* GitHub commits (last 1 year)
* GitHub followers
* npm/pip downloads
* Verified blog count
* Coding hours (from extension API calls)

All **raw metrics** saved in `metrics` table → server calculates score every 30 mins and updates `users.score`.

---

### **DB Schema (Supabase/Postgres)**

#### `users`

\| id (uuid) | username | role (`user`/`admin`) | country | university | github\_username | banned (bool) | created\_at |

#### `metrics`

\| id | user\_id | stars | followers | commits\_last\_year | package\_downloads | blog\_count\_verified | coding\_hours | last\_updated |

#### `blogs`

\| id | user\_id | url | verified (bool) | created\_at |

---

### **Score Calculation**

Runs in server cron (every 30 min):

```
score = (0.5 * stars) +
        (1 * blog_count_verified) +
        (0.01 * package_downloads) +
        (0.1 * commits_last_year) +
        (0.5 * followers) +
        (0.1 * total_coding_hours)
```

---

### **Weekend MVP Build Order**

1. **Supabase setup** → Tables for `users`, `metrics`, `blogs`
2. **Auth** → GitHub OAuth for users, secret-key Email/Password for admin
3. **Global Leaderboard page** (server-side fetch from Supabase)
4. **Profile page** for each user (SSR)
5. **Admin Dashboard basic UI** (view users, approve blogs)
6. **GitHub API integration** (GraphQL API for commits last year, stars, followers)
7. **Blog submission & verification** flow
8. **Score calculation job** (Supabase Edge Function or external cron hitting `/api/recalculate-scores`)
9. **Country & University Leaderboards** (filtered queries)

