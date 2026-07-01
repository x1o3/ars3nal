
## Footprinting

```sh
### scanning for services
> sudo nmap -sV -p 512,513,514 10.0.17.2

### login into services using r-login
> rlogin 10.0.17.2 -l htb-student

### checking authenticated users using rwho.
> rwho

### checking authenticated users using rusers.
> rusers -al 10.0.17.5
```

Once successfully logged in, we can also abuse the `rwho` command to list all interactive sessions on the local network by sending requests to the UDP port 513.

---