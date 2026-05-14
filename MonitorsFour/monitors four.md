idor user?token=0

admin:wonderful1

marcus:wonderful1

CVE-2025-24367-Cacti-PoC

curl -v http://host.docker.internal:2375/version

default api port on doker desktop

```
for i in $(seq 1 254); do (r=$(curl -sk --max-time 2 http://192.168.65.$i:2375/version 2>/dev/null); [ -n "$r" ] && echo "192.168.65.$i: $r") & done; wait
```
