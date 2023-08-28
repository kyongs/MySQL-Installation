# Run TPC-C on MySQL-5.7

**:warning: This guide is based on Ubuntu 18.04.**

## How to install MySQL 5.7

Building MySQL 5.7 from the source code enables you to customize build parameters, compiler optimizations, and installation location.

### Prerequisites

- libreadline

```bash
$ sudo apt-get install libreadline6 libreadline6-dev
```

If the package `libreadline6` is not available, download the newer version or ```libreadline-dev```:

```bash
$ sudo apt-get install libreadline7
```

- libaio

```bash
$ sudo apt-get install libaio1 libaio-dev
```

- etc.

```bash
$ sudo apt-get install build-essential cmake libncurses5 libncurses5-dev bison libssl-dev openssl
```

- If you are using VM or Virtualbox, please avoid installing MySQL with ```root```. Please make an additional user and resume the process.

### Build and install

1. Download the latest source code of [MySQL 5.7 Community Server](https://dev.mysql.com/downloads/mysql/5.7.html#downloads) using `wget`:

```bash
$ wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.33.tar.gz
```

2. Extract the `mysql-5.7.33.tar.gz` file:

```bash
$ tar -xvzf mysql-5.7.33.tar.gz
$ cd mysql-5.7.33
```

3. In MySQL 5.7, the Boost library is required to build MySQL. Therefore, download it first:

```bash
$ cmake -DDOWNLOAD_BOOST=ON -DWITH_BOOST=/path/to/download/boost -DCMAKE_INSTALL_PREFIX=/path/to/dir
```

If you already have the Boost library, update the path to the boost directory:

```bash
$ cmake -DWITH_BOOST=/path/to/boost -DCMAKE_INSTALL_PREFIX=/path/to/dir
```

4. Then, build and install the source code:
(8: # of cores in your machine, can be known by `grep 'cpu cores' /proc/cpuinfo | uniq` command.)

```bash
$ make -j8 install
```

5. Use `mysqld --initialize` to initialize the data directory, including the *mysql* database containing the initial MySQL grant tables that determine how users are permitted to connect to the server:
- `--datadir` : the path to the MySQL data directory
- `--basedir` : the path to the MySQL installation directory

```bash
$ ./bin/mysqld --initialize --user=mysql --datadir=/path/to/datadir --basedir=/path/to/basedir
```

6. Reset the root password:
- Open mysql server for connection.
```bash
$ ./bin/mysqld_safe --skip-grant-tables --datadir=/path/to/datadir
```

- Open an another terminal and connect as root.
```bash
$ ./bin/mysql -uroot

root:(none)> use mysql;

root:mysql> update user set authentication_string=password('yourPassword') where user='root';
root:mysql> flush privileges;
root:mysql> quit;

$ ./bin/mysql -uroot -p

root:mysql> set password = password('yourPassword');
root:mysql> quit;
```

7. Open `.bashrc` and add MySQL installation path to your path:

```bash
$ vi ~/.bashrc

export PATH=/path/to/basedir/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/basedir/lib/

$ source ~/.bashrc
```

8. Create a configuration file (`my.cnf`) for the MySQL server. For example, create a `my.cnf` in your local directory and copy the below texts to your `my.cnf`. You should modify the path `/path/to/datadir` to your local path. And if you separate the log device from the data device, uncomment `innodb_log_group_home_dir=/path/to/logdir/` and correct the path of the log directory:

```bash
$ vi my.cnf

#
# The MySQL database server configuration file
#
[client]
user    = root
port    = 3306
socket  = /tmp/mysql.sock

[mysql]
prompt  = \u:\d>\_

[mysqld_safe]
socket  = /tmp/mysql.sock

[mysqld]
# Basic settings
default-storage-engine = innodb
pid-file        = /path/to/datadir/mysql.pid
socket          = /tmp/mysql.sock
port            = 3306
datadir         = /path/to/datadir/
log-error       = /path/to/datadir/mysql_error.log

#
# Innodb settings
#
# Page size
innodb_page_size=16KB

# Buffer pool settings
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=8

# Transaction log settings
innodb_log_file_size=100M
innodb_log_files_in_group=2
innodb_log_buffer_size=32M

# Log group path (iblog0, iblog1)
# If you separate the log device, uncomment and correct the path
#innodb_log_group_home_dir=/path/to/logdir/

# Flush settings (SSD-optimized)
# 0: every 1 seconds, 1: fsync on commits, 2: writes on commits
innodb_flush_log_at_trx_commit=0
innodb_flush_neighbors=0
innodb_flush_method=O_DIRECT
```

For details on each variable, refer to [MySQL 5.7 Server System Variable](https://dev.mysql.com/doc/refman/5.7/en/server-system-variable-reference.html)

9. Shut down the MySQL server and restart it with `my.cnf`:

```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
$ ./bin/mysqld_safe --defaults-file=/path/to/my.cnf
```

10. You can shut down the server using the below command:

```bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
```



## Reference
- [Build and install the source code (5.7)](https://github.com/meeeejin/til/blob/master/mysql/build-and-install-the-source-code-5.7.md)
- [MySQL 5.7 Server System Variable Reference](https://dev.mysql.com/doc/refman/5.7/en/server-system-variable-reference.html)