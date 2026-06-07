# Fix WSL2 internet access when connected to Cisco AnyConnect VPN

This guide explains how to restore WSL2 network connectivity when Windows has internet access through Cisco AnyConnect VPN, but WSL2 cannot reach anything. This was tested with Windows 11 host and wsl2 container running Ubuntu 24.04.4 LTS

## 1. Find the VPN adapter in Windows

Open Windows PowerShell or Command Prompt and run:

```powershell
ipconfig /all
```

Look for the adapter associated with the VPN client.

For Cisco AnyConnect, look for an entry similar to:

- `Description . . . . . . . . . . . : Cisco AnyConnect Virtual Miniport Adapter for Windows x64`

Also look for the DNS servers listed under that adapter.

Example:

- `130.20.10.10`
- `130.10.10.10`

These are the VPN DNS servers.

## 2. Configure WSL to stop overwriting DNS

Inside WSL, open `/etc/wsl.conf`:

```bash
sudo nano /etc/wsl.conf
```

Make sure it contains:

```ini
[boot]
systemd=true

[user]
default=<your-linux-user>

[network]
generateResolvConf = false
```

Save the file.

## 3. Create a static resolv.conf in WSL

1. Inside WSL, create or edit `/etc/resolv.conf`:

```bash
sudo nano /etc/resolv.conf
```

2. Add the VPN DNS servers found from `ipconfig /all`.

Example:

```text
nameserver 130.10.10.10
nameserver 130.10.20.20
search pnl.gov
```

3. Save the file.

4. Restart WSL from Windows:

```powershell
wsl --shutdown
```

5. If you are running docker desktop restart docker desktop

6. Reopen the WSL distro and confirm the file was not overwritten:

```bash
cat /etc/resolv.conf
```

If the contents remain unchanged, DNS configuration is now static.

## 4. Test whether the problem is DNS or routing

1. Inside WSL, run:

```bash
ping -c 3 130.10.10.10
ping -c 3 8.8.8.8
ping -c 3 google.com
```

2. Interpret the results:

- If `google.com` fails but raw IPs work, the problem is DNS.
- If raw IPs such as `8.8.8.8` also fail, the problem is routing/networking, not DNS.

In this case, DNS was configured correctly, but all pings still failed, which showed that WSL2 traffic was not getting routed correctly while connected to the VPN.

## 5. Fix WSL2 routing with mirrored networking

1. Create a file named `.wslconfig` in the Windows home directory.

Example location:

```text
C:\Users\<windows-username>\.wslconfig
```

If the file does not exist, create it.

Add:

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
```

Notes:

- `networkingMode=mirrored` makes WSL use the host network more directly instead of the default WSL NAT network.
- `dnsTunneling=true` lets WSL send DNS queries through Windows.

2. After saving the file, restart WSL from Windows:

```powershell
wsl --shutdown
```

3. If docker desktop is running, restart docker desktop
   
4. Reopen WSL and test again:

```bash
ping -c 3 8.8.8.8
ping -c 3 google.com
```

If these now work, the VPN/WSL2 routing issue is resolved.

## 6. File location note for .wslconfig

The `.wslconfig` file belongs in the Windows home directory.

Example:

```text
C:\Users\D3X140\.wslconfig
```

This file does not need to exist ahead of time; create it manually if needed.

If `%USERPROFILE%` is available in Windows PowerShell, this command opens the correct file location:

```powershell
notepad $env:USERPROFILE\.wslconfig
```

If that environment variable is not convenient in the current shell, create `.wslconfig` directly in the Windows home directory instead.

## 7. Summary

Use this order:

1. Run `ipconfig /all` in Windows.
2. Look for the Cisco AnyConnect adapter.
3. Copy the DNS servers from that adapter into `/etc/resolv.conf` in WSL.
4. Disable automatic `resolv.conf` generation in `/etc/wsl.conf`.
5. Test connectivity from WSL.
6. If all pings still fail, create `C:\Users\<windows-user>\.wslconfig`.
7. Set `networkingMode=mirrored` and `dnsTunneling=true`.
8. Run `wsl --shutdown` and reopen WSL.
