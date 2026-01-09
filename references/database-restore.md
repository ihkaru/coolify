# Database Restoration Guide

This guide explains how to restore the `engine_backup.sql` file to your Coolify-managed MySQL database (`e_backend`).

## Prerequisites

- **Backup File:** `c:\coolify\engine_backup.sql`
- **Target Database:** `e_backend`
- **Container Name:** `jc4ocw4ok0ccwkk4k0gwg0s0`
- **Database User:** `root`
- **Database Password:** `Fikrizaki2`

> [!IMPORTANT]
> Ensure the backup file exists at the specified path before running commands.

## Restoration Steps

Run the following commands in **PowerShell**.

### 1. Verify Database Container is Running

```powershell
docker ps --filter "name=jc4ocw4ok0ccwkk4k0gwg0s0"
```

### 2. Create the Database (If it doesn't exist)

This command creates the `e_backend` database if it's missing.

```powershell
docker exec -i jc4ocw4ok0ccwkk4k0gwg0s0 mysql -u root -pFikrizaki2 -e "CREATE DATABASE IF NOT EXISTS e_backend;"
```

### 3. Restore the Backup

This command pipes the content of your SQL file directly into the database container.

> [!WARNING]
> This will **OVERWRITE** any existing data in the `e_backend` database.

```powershell
Get-Content c:\coolify\engine_backup.sql | docker exec -i jc4ocw4ok0ccwkk4k0gwg0s0 mysql -u root -pFikrizaki2 e_backend
```

## Troubleshooting

- **Error: `Table 'e_backend.xyz' doesn't exist`**: Make sure you ran Step 2 to create the database first.
- **Error: `Access denied for user 'root'`**: Verify the password is still `Fikrizaki2` in your Coolify database settings.
- **Error: `No such container`**: The container ID might have changed if the database was effectively redeployed. Check `docker ps` to find the new name.
