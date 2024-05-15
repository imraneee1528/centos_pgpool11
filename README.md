# Basis Instalations 
<pre>
  yum update -y &&  yum  -y install epel-release &&  yum -y install vim wget man tcpdump bash-completion net-tools yum-utils bind-utils sysstat dstat  lsof epel-release firewalld mlocate && yum -y install iftop htop nload telnet  && dnf install bash-completion virt-v2v-bash-completion.noarch yt-dlp-bash-completion.noarch nbdkit-bash-completion.noarch libnbd-bash-completion.noarch libguestfs-bash-completion.noarch fastfetch-bash-completion.noarch drbd-bash-completion.x86_64 -y &&  dnf  install git zip unzip && sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config && setenforce 0 && if ! grep -q "^vm.swappiness=10$" /etc/sysctl.conf; then echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf; fi && sudo sysctl -p

</pre>
# PG POOl Instalations 
<pre>
dnf install -y https://www.pgpool.net/yum/rpms/4.4/redhat/rhel-9-x86_64/pgpool-II-release-4.4-1.noarch.rpm 
dnf --enablerepo=crb install libmemcached-awesome-tools
dnf install -y pgpool-II-pg16 pgpool-II-pg16-extensions
</pre>
# Add In RC Local 
<pre>
vim /etc/rc.local
mkdir -p /var/run/postgresql
chown -R postgres:postgres /var/run/postgresql
/usr/bin/systemctl start pgpool.service
exit 0
</pre>
# keepalived Install Server 1
<pre>
dnf install keepalived -y
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.ori
vim /etc/keepalived/keepalived.conf
</pre>
<pre>
vrrp_instance VRRP1 {
    state MASTER
    interface eth0
    virtual_router_id 101
    priority 150
    unicast_src_ip 10.1.1.111/24
    unicast_peer {
        10.1.1.112/24
    }
    authentication {
        auth_type PASS
        auth_pass MNPSPBD_DR_BANGLAMEGH
    }
    virtual_ipaddress {
        10.1.1.110/24
    }
}
</pre>
<pre>
 systemctl restart keepalived.service 
 systemctl enable keepalived.service 
 systemctl status keepalived.service 
 ip a
</pre>
# keepalived Install Server 2
<pre>
dnf install keepalived -y
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.ori
vim /etc/keepalived/keepalived.conf
</pre>
<pre>
global_defs {
  router_id drpgdblb
}

vrrp_instance VRRP1 {
    state SLAVE
    interface eth0
    virtual_router_id 101
    priority 100
    unicast_src_ip 10.1.1.112/24
    unicast_peer {
        10.1.1.111/24
    }
    authentication {
        auth_type PASS
        auth_pass MNPSPBD_DR_BANGLAMEGH
    }
    virtual_ipaddress {
        10.1.1.110/24
    }
}
</pre>
<pre>
 systemctl restart keepalived.service 
 systemctl enable keepalived.service 
 systemctl status keepalived.service 
 ip a
</pre>
# Pg Pool Configurations 
<pre>
cp /etc/pgpool-II/pgpool.conf /etc/pgpool-II/pgpool.conf-bk
vim /etc/pgpool-II/pgpool.conf
</pre>
<pre>
#
# Listen Addresses and Port Settings
listen_addresses = '*'  # Listen on all available addresses
port = 5432  # Port number for PostgreSQL connections

# This is used when logging to stderr:
log_destination = 'stderr'
logging_collector = on
log_directory = '/var/log/pgpool_log'
log_filename = 'pgpool-%a.log'
log_truncate_on_rotation = on
log_rotation_age = 1d
log_rotation_size = 500MB 
unix_socket_directories = '/var/run/postgresql'
pcp_socket_dir = '/var/run/postgresql'
wd_ipc_socket_dir = '/var/run/postgresql'

backend_clustering_mode = 'streaming_replication'
# Backend Server Details
backend_hostname0 = 'drmnpspdatabaseserver1'  # Hostname/IP of the master 1 server
backend_port0 = 5432  # Port of the master server
backend_weight0 = 1  # Weight for the master (for write operations)

backend_hostname1 = 'drmnpspdatabaseserver2'  # Hostname/IP of themaster 2 server
backend_port1 = 5432  # Port of the master server
backend_weight0 = 1  # Weight for the master (for write operations)

# Load Balancing
load_balance_mode = 'ON'  # Enable load balancing

# Replication Responsibility
master_slave_mode = 'ON'  # Enable master-slave mode
master_slave_sub_mode = 'stream'  # Use streaming replication

# Streaming Checks
sr_check_period = 10  # Check interval for streaming replication status
sr_check_user = 'monit'  # User for monitoring
sr_check_password = 'monitor'  # Password for monitoring
sr_check_database = 'eic8ohphiQu#eechi3te'  # Database for monitoring
delay_threshold = 10240  # Threshold for replication delay

# Client Authentication
#allow_clear_text_frontend_auth = 'ON'  # Allow clear-text authentication
enable_pool_hba = off
allow_clear_text_frontend_auth = on
pool_passwd = ''
# Connection Pooling
num_init_children = 1000 # Number of initial pooler child processes
max_pool = 4  # Maximum number of connections per pooler child process
#child_life_time = 1200  # Maximum time (in seconds) a pooler child process can live
#child_idle_timeout = 120  # Idle timeout (in seconds) for pooler child processes

# Connection Pooling Behavior
#connection_life_time = 0  # Maximum time (in seconds) a connection can be reused, 0 means unlimited
#client_idle_limit = 0  # Maximum time (in seconds) a client can remain idle, 0 means no limit
#connection_cache = 'on'  # Enable connection caching for reuse

# Query Cache
#memory_cache_enabled = 'ON'  # Enable memory-based query cache
#memqcache_method = 'shmem'  # Method for memory-based query cache
#memqcache_total_size = 1024MB  # Total size of query cache (in MB)
#memqcache_max_num_cache = 100000  # Maximum number of cached queries

# Health Check
health_check_period = 10  # Health check interval (in seconds)
health_check_timeout = 20  # Health check timeout (in seconds)
health_check_user = 'monit'  # User for health checking
health_check_database = 'monitor' # Database for health checking 
health_check_password = 'eic8ohphiQu#eechi3te'  # Password for health checking

# Other Settings
#serialize_accept = 'off'  # Disable accept serialization
</pre>
# Postgresql Client Install
<pre>
  sudo vi /etc/yum.repos.d/pgdg-16.repo
</pre>
<pre>
[pgdg16]
name=PostgreSQL 16 for RHEL/CentOS $releasever - $basearch
baseurl=https://download.postgresql.org/pub/repos/yum/16/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
</pre>
<pre>
dnf install postgresql16 -y
systemctl start   pgpool.service
systemctl enable  pgpool.service
systemctl status  pgpool.service
tail -f /var/log/pgpool_log/pgpool-Wed.log
</pre>
