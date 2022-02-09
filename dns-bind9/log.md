# Log

how to setup bind9 log file

## crate config file

```sh
vi /var/named/chroot/etc/named.conf.options
```

```conf
logging {
	channel transfers {
		file "/var/log/bind/transfers" versions 3 size 10M;
		print-time yes;
		severity info;
	};
	channel notify {
		file "/var/log/bind/notify" versions 3 size 10M;
		print-time yes;
		severity info;
	};
	channel dnssec {
		file "/var/log/bind/dnssec" versions 3 size 10M;
		print-time yes;
		severity info;
	};
	channel query {
		file "/var/log/bind/query" versions 5 size 10M;
		print-time yes;
		severity info;
	};
	channel general {
		file "/var/log/bind/general" versions 3 size 10M;
	print-time yes;
	severity info;
	};
	channel slog {
		syslog security;
		severity info;
	};
	category xfer-out { transfers; slog; };
	category xfer-in { transfers; slog; };
	category notify { notify; };

	category lame-servers { general; };
	category config { general; };
	category default { general; };
	category security { general; slog; };
	category dnssec { dnssec; };

	// category queries { query; };
};
```

## edit named.conf

```sh
vi named.conf
```

```conf
include "/etc/named.conf.options";
```

```sh
mkdir -p /var/named/chroot/var/log/bind/ # because i use chroot
# or
mkdir -p /var/log/bind/

chown -R named:named /var/named/chroot/var/log/bind/

/etc/init.d/named restart
```

## test

```sh
tail -f /var/named/chroot/var/log/bind/general
```
