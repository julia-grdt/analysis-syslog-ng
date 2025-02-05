# Installations
brew install coreutils
brew install syslog-ng

# Config file
/usr/local/etc/syslog-ng/syslog-ng.conf

## Incorrect mac path

@version: 4.8
@include "scl.conf"

source s_network {
  network(
    transport("udp")
    port(514)
    # NOTE: Secure logging requires this flag to be set
    flags(store-raw-message)
  );
};

template t_slog {
  template("$(slog -k /Users/anitazut/POC/host.key -m /not/a/path/mac.dat $RAWMSG)\n");
};

destination d_local {
  file("/var/log/messages.slog" template(t_slog));
};

log {
  source(s_network);
  destination(d_local);
};

## Result
Still runs without errors when starting up syslog-ng, but can not verify as the path doesnt exist so the mac file doesn't exist.

$ sudo pkill -9 syslog-ng
$ 
$ rm -rf POC
$ mkdir POC
$ cd POC
$ sudo slogkey --master-key master.key
$ sudo slogkey -d master.key 6c:b1:33:9f:73:31 X9WRXHV6PC host.key
$ scp host.key ~/Documents/copy_host.key
$ ls
host.key    master.key
$ 
$ sudo rm -f /var/log/messages.slog
$ sudo touch /var/log/messages.slog
$ 
$ sudo cat /var/log/messages.slog
$ 
$ sudo syslog-ng
$ ps aux | grep syslog-ng
root             74423  79,9  0,1 34894372  40864   ??  Ss    4:18pm   0:00.55 syslog-ng
root             74422   0,1  0,0 34286332    800   ??  S     4:18pm   0:00.00 supervising syslog-ng
Julia            74425   0,0  0,0 410733264   1488 s003  S+    4:18pm   0:00.00 grep syslog-ng
$ sudo lsof -i :514
COMMAND     PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
syslog-ng 74423 root   24u  IPv4 0xdb78c7d2f704071f      0t0  UDP *:syslog
$ 
$ echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514
$ 
<13>Test log message
$ sudo cat /var/log/messages.slog
AAAAAAAAAAA=:T3moQJwmHIHGBvaH0qm/lyfBbHNFLmkzh5njhTk+nfZA/WbfiZEbXyb8BasMK9tw
$ 
$ sudo chmod 666 /var/log/messages.slog
$ 
$ slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat  /var/log/messages.slog out.log

ERROR: Invalid path or non existing regular file: /Users/anitazut/POC/mac.dat

Gebruik:
  slogverify [OPTIE?] INPUTLOG OUTPUTLOG [COUNTER] - Log archive verification

Hulpopties:
  -h, --help                   Deze hulptekst tonen

Programmaopties:
  -i, --iterative              Iterative verification
  -k, --key-file=FILE          Initial host key file
  -m, --mac-file=FILE          Current MAC file
  -p, --prev-key-file=FILE     Previous host key file in iterative mode
  -r, --prev-mac-file=FILE     Previous MAC file in iterative mode

## Correct mac path
@version: 4.8
@include "scl.conf"

source s_network {
  network(
    transport("udp")
    port(514)
    # NOTE: Secure logging requires this flag to be set
    flags(store-raw-message)
  );
};

template t_slog {
  template("$(slog -k /Users/anitazut/POC/host.key -m /Users/anitazut/POC/mac.dat $RAWMSG)\n");
};

destination d_local {
  file("/var/log/messages.slog" template(t_slog));
};


log {
  source(s_network);
  destination(d_local);
};

## Results

### Start syslog-ng with incorrect host.key and incorrect mac.dat
sudo pkill -9 syslog-ng

rm -rf POC
mkdir POC
cd POC
sudo slogkey --master-key master.key
touch host.key
touch mac.dat
scp host.key ~/Documents/copy_host.key


sudo rm -f /var/log/messages.slog
sudo touch /var/log/messages.slog

sudo cat /var/log/messages.slog

sudo syslog-ng
ps aux | grep syslog-ng
sudo lsof -i :514

echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514

sudo cat /var/log/messages.slog
sudo chmod 666 /var/log/messages.slog

slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log

cd ..

### Result
Still runs with one error and one warning when starting up, but can not verify as the key file can not be read.

$ sudo pkill -9 syslog-ng
$ 
$ rm -rf POC
$ mkdir POC
$ cd POC
$ sudo slogkey --master-key master.key
$ touch host.key
$ touch mac.dat
$ scp host.key ~/Documents/copy_host.key
$ 
$ 
$ sudo rm -f /var/log/messages.slog
$ sudo touch /var/log/messages.slog
$ 
$ sudo cat /var/log/messages.slog
$ 
$ sudo syslog-ng
[2025-01-18T14:47:26.778544] [SLOG] ERROR: Cannot read from key file;
[2025-01-18T14:47:26.778544] [SLOG] WARNING: Template parsing failed, key file not found or invalid. Reverting to clear text logging.;
$ ps aux | grep syslog-ng
root              1250  85,3  0,1 34738720  40508   ??  Ss    2:47pm   0:00.54 syslog-ng
Julia             1253   0,0  0,0 410724048   1344 s000  S+    2:47pm   0:00.00 grep syslog-ng
root              1247   0,0  0,0 34286332    812   ??  S     2:47pm   0:00.00 supervising syslog-ng
$ sudo lsof -i :514
COMMAND    PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
syslog-ng 1250 root   23u  IPv4 0xeca8e9512b221080      0t0  UDP *:syslog
$ 
$ echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514
$ 
$ sudo cat /var/log/messages.slog
<13>Test log message
$ sudo chmod 666 /var/log/messages.slog
$ 
$ slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log
[SLOG] INFO: Reading key file; name='/Users/anitazut/Documents/copy_host.key'
[SLOG] ERROR: Cannot read from key file;
[SLOG] ERROR: Unable to read host key; file='/Users/anitazut/Documents/copy_host.key'

### Start syslog-ng with empty mac.dat
sudo pkill -9 syslog-ng

rm -rf POC
mkdir POC
cd POC
sudo slogkey --master-key master.key
sudo slogkey -d master.key 6c:b1:33:9f:73:31 X9WRXHV6PC host.key
touch mac.dat
scp host.key ~/Documents/copy_host.key


sudo rm -f /var/log/messages.slog
sudo touch /var/log/messages.slog

sudo cat /var/log/messages.slog

sudo syslog-ng
ps aux | grep syslog-ng
sudo lsof -i :514

echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514

sudo cat /var/log/messages.slog

sudo chmod 666 /var/log/messages.slog

slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log

cat out.log

cd ..

### Result
Still runs with one error, logs can be verified.

$ sudo pkill -9 syslog-ng
$ 
$ rm -rf POC
$ mkdir POC
$ cd POC
$ sudo slogkey --master-key master.key
$ sudo slogkey -d master.key 6c:b1:33:9f:73:31 X9WRXHV6PC host.key
$ touch mac.dat
$ scp host.key ~/Documents/copy_host.key
$ 
$ 
$ sudo rm -f /var/log/messages.slog
$ sudo touch /var/log/messages.slog
$ 
$ sudo cat /var/log/messages.slog
$ 
$ sudo syslog-ng
[2025-01-18T14:49:19.795040] [SLOG] ERROR: Cannot read MAC file;
$ ps aux | grep syslog-ng
root              1649  82,6  0,1 34607652  40200   ??  Ss    2:49pm   0:00.54 syslog-ng
Julia             1651   0,0  0,0 410724048   1344 s000  S+    2:49pm   0:00.00 grep syslog-ng
root              1648   0,0  0,0 34286332    824   ??  S     2:49pm   0:00.00 supervising syslog-ng
$ sudo lsof -i :514
COMMAND    PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
syslog-ng 1649 root   23u  IPv4 0x87a390556a1e64bf      0t0  UDP *:syslog
$ 
$ echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514
$ 
$ sudo cat /var/log/messages.slog
AAAAAAAAAAA=:HpsKHmbrU/OUmRSvuow8noIu6m8zAHKVHicDLOElI8R5rUbBrfeOZ+cJFCaLJrwA
$ 
$ sudo chmod 666 /var/log/messages.slog
$ 
$ slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log
[SLOG] INFO: Reading key file; name='/Users/anitazut/Documents/copy_host.key'
[SLOG] INFO: Reading MAC file; name='/Users/anitazut/POC/mac.dat'
[SLOG] INFO: MAC successfully loaded;
[SLOG] INFO: Number of lines in file; number='1'
[SLOG] INFO: Restoring and verifying log entries; buffer size='1000'
[SLOG] INFO: All entries recovered successfully;
[SLOG] Aggregated MAC matches. Log contains all expected log messages.;
$ 
$ cat out.log
00000000000000000000: <13>Test log message
$ 
$ cd ..

### Start syslog-ng with no mac.dat
sudo pkill -9 syslog-ng

rm -rf POC
mkdir POC
cd POC
sudo slogkey --master-key master.key
sudo slogkey -d master.key 6c:b1:33:9f:73:31 X9WRXHV6PC host.key
scp host.key ~/Documents/copy_host.key
ls

sudo rm -f /var/log/messages.slog
sudo touch /var/log/messages.slog

sudo cat /var/log/messages.slog

sudo syslog-ng
ps aux | grep syslog-ng
sudo lsof -i :514

echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514

sudo cat /var/log/messages.slog

sudo chmod 666 /var/log/messages.slog

slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log

cat out.log

cd ..

### Result
Runs with no errors

$ sudo pkill -9 syslog-ng
$ 
$ rm -rf POC
$ mkdir POC
$ cd POC
$ sudo slogkey --master-key master.key
$ sudo slogkey -d master.key 6c:b1:33:9f:73:31 X9WRXHV6PC host.key
$ scp host.key ~/Documents/copy_host.key
$ ls
host.key    master.key
$ 
$ sudo rm -f /var/log/messages.slog
$ sudo touch /var/log/messages.slog
$ 
$ sudo cat /var/log/messages.slog
$ 
$ sudo syslog-ng
$ ps aux | grep syslog-ng
root              1708  84,6  0,1 34886180  40820   ??  Ss    2:49pm   0:00.55 syslog-ng
Julia             1710   0,0  0,0 410724048   1344 s000  S+    2:49pm   0:00.00 grep syslog-ng
root              1707   0,0  0,0 34286332    800   ??  S     2:49pm   0:00.00 supervising syslog-ng
$ sudo lsof -i :514
COMMAND    PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
syslog-ng 1708 root   23u  IPv4 0x19d8c1ac2e9bb737      0t0  UDP *:syslog
$ 
$ echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514
$ 
$ sudo cat /var/log/messages.slog
AAAAAAAAAAA=:raiQe79gYJNQwN5+qXqbpQ7E0xXXgY1fKVHFLFWKt/YgIS4UzWQaXzKBehLmJwTW
$ 
$ sudo chmod 666 /var/log/messages.slog
$ 
$ slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log
[SLOG] INFO: Reading key file; name='/Users/anitazut/Documents/copy_host.key'
[SLOG] INFO: Reading MAC file; name='/Users/anitazut/POC/mac.dat'
[SLOG] INFO: MAC successfully loaded;
[SLOG] INFO: Number of lines in file; number='1'
[SLOG] INFO: Restoring and verifying log entries; buffer size='1000'
[SLOG] INFO: All entries recovered successfully;
[SLOG] Aggregated MAC matches. Log contains all expected log messages.;
$ 
$ cat out.log
00000000000000000000: <13>Test log message
$ 
$ cd ..



### Start syslog-ng with incorrect mac.dat
sudo pkill -9 syslog-ng

rm -rf POC
mkdir POC
cd POC
sudo slogkey --master-key master.key
sudo slogkey -d master.key 6c:b1:33:9f:73:31 X9WRXHV6PC host.key
touch mac.dat
echo "thisisnotacorrectmacvalue" > mac.dat
cat mac.dat
scp host.key ~/Documents/copy_host.key


sudo rm -f /var/log/messages.slog
sudo touch /var/log/messages.slog

sudo cat /var/log/messages.slog

sudo syslog-ng
ps aux | grep syslog-ng
sudo lsof -i :514

echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514

sudo cat /var/log/messages.slog

sudo chmod 666 /var/log/messages.slog

cat mac.dat

slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log

cat out.log

cd ..

### Result
Still works as it will just create a new MAC file with a new correct value.

$ sudo pkill -9 syslog-ng
$ 
$ rm -rf POC
$ mkdir POC
$ cd POC
$ sudo slogkey --master-key master.key
$ sudo slogkey -d master.key 6c:b1:33:9f:73:31 X9WRXHV6PC host.key
$ touch mac.dat
$ echo "thisisnotacorrectmacvalue" > mac.dat
$ cat mac.dat
thisisnotacorrectmacvalue
$ scp host.key ~/Documents/copy_host.key
$ 
$ 
$ sudo rm -f /var/log/messages.slog
$ sudo touch /var/log/messages.slog
$ 
$ sudo cat /var/log/messages.slog
$ 
$ sudo syslog-ng
[2025-01-23T14:07:03.851592] [SLOG] ERROR: $(slog) parsing failed, invalid size of MAC file;
$ ps aux | grep syslog-ng
root             56439  70,8  0,1 34606632  40252   ??  Ss    2:07pm   0:00.42 syslog-ng
root             56438   0,2  0,0 34286332    800   ??  S     2:07pm   0:00.00 supervising syslog-ng
Julia            56442   0,0  0,0 410724048   1344 s000  S+    2:07pm   0:00.00 grep syslog-ng
$ sudo lsof -i :514
COMMAND     PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
syslog-ng 56439 root   23u  IPv4 0xcfb6a825cb8f597e      0t0  UDP *:syslog
$ 
$ echo "<13>Test log message" | gtimeout 1 nc -u 127.0.0.1 514
$ 
$ sudo cat /var/log/messages.slog
AAAAAAAAAAA=:61rFkAdnNjcEYwRShAldn5plcecv7JW4mv2a4VrvYXXBu0duU9cWqsBBSKB8sk6r
$ 
$ sudo chmod 666 /var/log/messages.slog
$ 
$ cat mac.dat
;?k?o???4??C?E
              d?4?.??G???n߷$ 
$ slogverify -k  ~/Documents/copy_host.key -m /Users/anitazut/POC/mac.dat /var/log/messages.slog  out.log
[SLOG] INFO: Reading key file; name='/Users/anitazut/Documents/copy_host.key'
[SLOG] INFO: Reading MAC file; name='/Users/anitazut/POC/mac.dat'
[SLOG] INFO: MAC successfully loaded;
[SLOG] INFO: Number of lines in file; number='1'
[SLOG] INFO: Restoring and verifying log entries; buffer size='1000'
[SLOG] INFO: All entries recovered successfully;
[SLOG] Aggregated MAC matches. Log contains all expected log messages.;
$ 
$ cat out.log
00000000000000000000: <13>Test log message
$ 
$ cd ..
