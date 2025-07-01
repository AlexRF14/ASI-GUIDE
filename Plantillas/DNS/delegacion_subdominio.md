## Now how could I set it up if I wanted my Rocky machine to have the delegation of the  subdomain 'examen.net' like 'uni.examen.net'

To **delegate the subdomain `uni.examen.net` to your Rocky machine**, you need to update the parent (`examen.net`) zone on your master server (Ubuntu) so that all queries for `uni.examen.net` (and anything below it, like `host.uni.examen.net`) are referred to the Rocky server as the authority.

Here’s how to do it:

## **1. On the Master (Ubuntu, authoritative for `examen.net`)**

**Edit your `db.examen.net` zone file** (or wherever your `examen.net` zone is defined).

**Add NS records for the subdomain:**
```dns
uni    IN  NS  rocky1.uni.examen.net.
uni    IN  NS  rocky2.uni.examen.net.
```
- Replace `rocky1.uni.examen.net.` and `rocky2.uni.examen.net.` with the actual hostnames of your Rocky server(s).  
- If you only have one Rocky server, you can specify just one NS record.

**Add A records for those hostnames** (if they are within your domain):
```dns
rocky1.uni    IN  A   192.168.52.129
```
- Or, if you want to use a name outside the `uni.examen.net` subdomain, just make sure it resolves to the correct IP.

**Example snippet:**
```dns
; Delegation for uni.examen.net
uni    IN  NS  rocky1.uni.examen.net.

rocky1.uni    IN  A   192.168.52.129
```

**DO NOT** create a zone file for `uni.examen.net` on the master. The NS records in the parent zone are what delegates authority[1][2][3].

## **2. On the Rocky Server (authoritative for `uni.examen.net`)**

**Set up a new zone file for `uni.examen.net`:**

In `/etc/named.conf` (or an included file), add:
```conf
zone "uni.examen.net" IN {
    type master;
    file "/var/named/uni.examen.net.zone";
    allow-transfer { ... };  // as needed
    allow-query { ... };     // as needed
};
```

**Create the zone file `/var/named/uni.examen.net.zone`:**
```dns
$TTL 3600
@   IN  SOA rocky1.uni.examen.net. admin.uni.examen.net. (
        2025070101 ; Serial
        3600       ; Refresh
        1800       ; Retry
        1209600    ; Expire
        3600 )     ; Minimum

    IN  NS  rocky1.uni.examen.net.

rocky1  IN  A   192.168.52.129
; Add other records as needed
```

**Restart/reload BIND on Rocky:**
```bash
sudo systemctl reload named
```

## **3. How It Works**

- When a resolver asks the master for `host.uni.examen.net`, the master returns the NS records for `uni.examen.net`, pointing to the Rocky server.
- The resolver then queries the Rocky server for the actual answer.
- All records under `uni.examen.net` are now managed independently by the Rocky server[1][2][3].

## **Summary Table**

| Server         | Role                     | Zone file(s) to edit             | Key config                |
|----------------|--------------------------|----------------------------------|---------------------------|
| Ubuntu (Master)| Parent (`examen.net`)    | `db.examen.net`                  | Add NS delegation for `uni` |
| Rocky (Slave)  | Subdomain (`uni.examen.net`) | `/var/named/uni.examen.net.zone` | Set up as master for subdomain |

**References:**  
- Add NS records for the subdomain in the parent zone to delegate authority[1][2][3].
- Do not create a subdomain zone on the parent; only on the delegated server.  
- Ensure the delegated server is reachable and authoritative for the subdomain.

