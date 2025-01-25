# PostgreSQL 16 Replication Setup Guide

## 🖥️ Network Configuration
- **Primary Server**: 192.168.1.2 (Port 5432)
- **Standby Server**: 192.168.1.3 (Port 5432)

## 📋 Prerequisite: PostgreSQL 16 Installation
Install PostgreSQL 16 on both server and standby machines.

## 🔧 Primary Server Configuration

### 1. Edit `postgresql.conf`
```conf
listen_addresses = '*'
max_wal_senders = 5
wal_level = replica
archive_mode = on
archive_command = 'copy %p C:/Program Files/PostgreSQL/16/data/archive/%f'
hot_standby = on
```

### 2. Create Replication Role
```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'your_secure_password';
```

### 3. Configure `pg_hba.conf`
Add replication access line:
```conf
host    replication     replicator    192.168.1.0/24    md5
```

## 🔄 Standby Server Setup

### 1. Update `pg_hba.conf`
Add primary server connection details.

### 2. Prepare Data Directory
- Clear existing data folder
- Run base backup:
```conf
./pg_basebackup -h 192.168.1.2 -U replicator -D "C:\Program Files\PostgreSQL\16\data" -Fp -Xs -P -p 5432
```


### 3. Configure `postgresql.auto.conf`
```conf
primary_conninfo = 'host=192.168.1.2 port=5432 user=replicator password=your_secure_password'
```

### 4. Create Standby Signal
Create an empty file named `standby.signal` in the data directory.

## 🔍 Verification Steps for PostgreSQL Replication

### 1. Service Startup
- Start PostgreSQL service on both primary and standby servers

### 2. Replication Status Check
Run the following SQL query on the primary server:
```sql
SELECT * FROM pg_stat_replication;
```
it should return 1 row

## ⚠️ Next step
- automated failover

## 🔍 Troubleshooting
- verify firewall rules and ensure you open port 5432 in both pcs
- Check logs for connection issues
- Verify network connectivity
- Ensure matching PostgreSQL versions


## ✍️ Author
Omeiri Abdellah

