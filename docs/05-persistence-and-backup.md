# Task 5 — Persistence & Backup Plan

## How data persists
MySQL data lives in the `db_data` named volume, mounted at `/var/lib/mysql`. It survives container
restarts and `docker compose down` / `up`. Never use `docker compose down -v`: it deletes the volume.

## What we back up
A `mysqldump` of the `bookstore` database (full schema + data, as portable SQL).
We do not back up raw volume files directly, because a SQL dump is portable across
MySQL versions and easy to inspect or move.

## When
Daily, at a low-traffic hour (example: 2 AM server time).

## Where
Initially on the same VM's disk, in a `backups/` folder outside the Docker
volume. For a production setup we would add a step to upload each dump to
object storage (Azure Blob Storage), so a full VM loss doesn't also destroy the backups.
Backup files are excluded from Git (`.gitignore`).

## Retention
Keep the last 7 daily dumps and delete anything older, to avoid unbounded disk growth.

## Automation (planned cron job)

    0 2 * * * cd /home/azureuser/theepicbook && docker compose exec -T database sh -c 'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" bookstore' > backups/bookstore-$(date +\%Y\%m\%d).sql
    0 3 * * * find /home/azureuser/theepicbook/backups -name "*.sql" -mtime +7 -delete

## Restore procedure
1. List available backups: `ls -lh backups/`
2. Restore the chosen file:

    cat backups/<chosen-file>.sql | docker compose exec -T database sh -c 'mysql -uroot -p"$MYSQL_ROOT_PASSWORD" bookstore'

3. Verify the data by querying the restored table (e.g. row count).

## Manual test performed
1. Confirmed baseline count: 54 books.
2. Deleted book id 1 ("28 Summers").
3. Confirmed count dropped to 53, proving real data loss.
4. Restored from a `mysqldump` backup taken beforehand.
5. Confirmed count returned to 54, and book id 1 ("28 Summers") was present again.

## Tested
Yes, manually, on 2026-10-06. See screenshots (volume inspect,
before/delete/restore counts, container-restart persistence check).
