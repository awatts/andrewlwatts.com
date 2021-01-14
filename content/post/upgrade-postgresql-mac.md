---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Upgrading Postgresql on macOS"
subtitle: ""
summary: "Upgrading Postgres versions on macOS when using the EnterpriseDB installer
 can be confusing. This is how to do it."
authors: []
tags: []
categories: []
date: 2021-01-14T11:12:01-05:00
lastmod: 2021-01-14T11:12:01-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
# image:
#   caption: ""
#   focal_point: ""
#   preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Many Mac users are perfectly well served by install PostgreSQL through [homebrew](https://brew.sh/)
and letting it automatically upgrade and migrate your data. For work though, I have to
be sure I'm running the same version on my local dev machine as our dev and prod servers.

I keep things from changing underneath me by using the installers provided by
[EDB](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads), which are
free (they make money on support contracts and an enterprise version of PG).

Upgrading is something I have to do just infrequently enough that I forget the details
and have to figure it out again each time, so I'm writing this for with hopes that it
also can benefit other people.

Starting with the assumption that we're upgrading from 11.x to 12.x, but subsitute
the older version and newer version on your system as necessary.

- Install new version of EDB Postgres
- Stop it with: `sudo launchctl unload /Library/LaunchDaemons/com.edb.launchd.postgresql-12.plist`
- `sudo su postgres` and then `cd /Library/PostgreSQL/11/data/` (you need to run the upgrade command in a directory postgres user has write perms in)

  - Edit `/Library/PostgreSQL/11/data/pg_hba.conf` and change local auth lines from *md5* to *trust* (this is temporary because the upgrade script doesn't pass in passwords)
  - Do same for 12
  - Check that the upgrade script is going to work:

```bash
  /Library/PostgreSQL/12/bin/pg_upgrade \
      --old-datadir=/Library/PostgreSQL/11/data \
      --new-datadir=/Library/PostgreSQL/12/data
      --old-bindir=/Library/PostgreSQL/11/bin \
      --new-bindir=/Library/PostgreSQL/12/bin \
      --old-options='-c config_file=/Library/PostgreSQL/11/data/postgresql.conf' \
      --new-options='-c config_file=/Library/PostgreSQL/12/data/postgresql.conf' \
      --check
```

- Stop old server with: `sudo launchctl unload /Library/LaunchDaemons/com.edb.launchd.postgresql-11.plist`
- Run upgrade script without `--check`

  ```bash
  /Library/PostgreSQL/12/bin/pg_upgrade \
      --old-datadir=/Library/PostgreSQL/11/data \
      --new-datadir=/Library/PostgreSQL/12/data
      --old-bindir=/Library/PostgreSQL/11/bin \
      --new-bindir=/Library/PostgreSQL/12/bin \
      --old-options='-c config_file=/Library/PostgreSQL/11/data/postgresql.conf' \
      --new-options='-c config_file=/Library/PostgreSQL/12/data/postgresql.conf'
    ```

- Edit `/Library/PostgreSQL/12/data/postgresql.conf` and change port to 5432
- Edit `/Library/PostgreSQL/11/data/postgresql.conf` and change port to 5433 (optional;
you can also just uninstall it by running the uninstall app at `/Library/PostgreSQL/11/uninstall-postgresql.app`)

- Edit `/Library/PostgreSQL/12/data/pg_hba.conf` and change local auth lines from *trust* to *md5*
- Do same for 11 (if you aren't uninstalling it)

- Run `./analyze_new_cluster.sh` to optimize the new databases

- Start the new server with: `sudo launchctl unload /Library/LaunchDaemons/com.edb.launchd.postgresql-12.plist`
