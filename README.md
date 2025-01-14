# PostgreSQL High Availability Cluster Setup
> Automated failover configuration for PostgreSQL databases on Windows servers

## Overview
This repository provides comprehensive documentation and configuration files for setting up a high-availability PostgreSQL cluster with automatic failover between two Windows servers. The setup uses Patroni for cluster management and supports streaming replication.

## 🎯 Features
- PostgreSQL streaming replication between primary and standby servers
- Automatic failover management using Patroni
- etcd for distributed configuration
- Optional load balancer configuration
- Comprehensive security considerations
- Backup strategy implementation

## 🛠 Architecture
- **Server A (Primary)**: 192.168.1.2
- **Server B (Standby)**: 192.168.1.3
- **Port Configuration**:
  - PostgreSQL: 5432
  - Patroni REST API: 8008
  - etcd: 2379

## 📋 Prerequisites
- Windows Server (both nodes)
- PostgreSQL for Windows
- Python
- etcd
- Patroni
- Administrative privileges

## 🚀 Installation Steps

### 1. PostgreSQL Installation
```bash
# Download and install PostgreSQL on both servers
# Set the following during installation:
- Installation Directory: C:\Program Files\PostgreSQL\<version>
- Data Directory: C:\Program Files\PostgreSQL\<version>\data
- Port: 5432 (default)
```

### 2. Streaming Replication Configuration

#### Primary Server (192.168.1.2)
1. Edit `postgresql.conf`:
```conf
wal_level = replica
max_wal_senders = 5
listen_addresses = '192.168.1.2,localhost'
synchronous_commit = on
```

2. Configure `pg_hba.conf`:
```conf
host replication all 192.168.1.3/32 md5
```

#### Standby Server (192.168.1.3)
1. Initialize standby cluster:
```bash
pg_basebackup -h 192.168.1.2 -D "C:\Program Files\PostgreSQL\<version>\data" -U replication_user -P -v -X stream
```

2. Configure recovery settings:
```conf
standby_mode = 'on'
primary_conninfo = 'host=192.168.1.2 port=5432 user=replication_user password=your_password'
trigger_file = 'C:\Program Files\PostgreSQL\<version>\data\failover_trigger_file'
```

### 3. Patroni Setup

#### Install and Configure etcd
1. Download etcd from GitHub releases
2. Extract and configure on each node

#### Install Patroni
```bash
pip install patroni[etcd]
```

#### Configure Patroni
Create `patroni.yml`:
```yaml
scope: my_cluster
name: server_a

restapi:
  listen: 192.168.1.2:8008

etcd:
  hosts: http://192.168.1.3:2379

postgresql:
  data_dir: C:\Program Files\PostgreSQL\<version>\data
  listen: 192.168.1.2:5432
  connect_address: 192.168.1.2:5432
  parameters:
    wal_level: replica
    max_wal_senders: 5
    hot_standby: 'on'
    synchronous_commit: on
    max_replication_slots: 5
    shared_preload_libraries: 'pg_stat_statements, replication_slots'
  authentication:
    replication:
      username: replication_user
      password: your_password
```

## 🔒 Security Configuration
1. Configure firewall rules for required ports
2. Implement SSL for inter-server communication
3. Use strong passwords for all services

## 💾 Backup Strategy
- Regular base backups
- WAL archiving
- Offsite backup storage
- Periodic restoration testing

## 🧪 Testing Failover
1. Verify replication status
2. Simulate primary server failure
3. Confirm automatic standby promotion
4. Test primary server recovery

## 📊 Monitoring
- Implement Prometheus and Grafana for monitoring
- Track replication status
- Monitor cluster health
- Set up alerting for critical events

## 🤝 Contributing
Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ✍️ Author
Omeiri Abdellah

## 📧 Support
For support and questions, please open an issue in the repository.
