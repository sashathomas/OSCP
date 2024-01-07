# Forwarding & Tunneling

**Socat port forward**

```js
socat TCP-LISTEN:4141,fork TCP:<remote IP>:<remote port>
```

Remote port is now accessible at **Local IP** and **Local Port**

------

## Netcat Port Scan

```js
for i in $(seq 1 254); do nc -zv -w 1 172.16.50.$i 445; done
```








