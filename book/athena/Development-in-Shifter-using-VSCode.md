# Development in Shifter using VSCode
https://docs.nersc.gov/development/containers/shifter/examples/#development-in-shifter-using-vscode

### Step 1
_At NERSC_, create a script `$HOME/.local/bin/run-shifter` that looks like this

```bash
#!/bin/sh 
export XDG_RUNTIME_DIR="${TMPDIR:-/tmp}/`whoami`/run" 
exec shifter --image="$1"
```

This is necessary since VS-Code tries to access the `$XDG_RUNTIME_DIR`. At NERSC, `$XDG_RUNTIME_DIR` points to `/run/user/YOUR_UID` by default, which is not accessible from within Shifter container instances, so we need to override the default location.

### Step 2

In your `"$HOME/.ssh/config"` on your _local system_, add something like

```config
Host someimage~*   
  RemoteCommand ~/.local/bin/run-shifter someorg/someimage:latest   
  RequestTTY yes  
  
Host otherimage~*   
  RemoteCommand ~/.local/bin/run-shifter someorg/someimage:latest   
  RequestTTY yes  
  
Host perlmutter*.nersc.gov   
  IdentityFile ~/.ssh/nersc  
  
Host perlmutter someimage~perlmutter otherimage~perlmutter   
  HostName perlmutter.nersc.gov   
  User YOUR_NERSC_USERNAME
```

Test whether this works by running `ssh someimage~perlmutter` _on your local system_. This should drop you into an SSH session running inside of an instance of the `someorg/someimage:latest` container image at NERSC.

### Step 3

In your VS-Code settings _on your local system_, set

`"remote.SSH.enableRemoteCommand": true`

### Step 4

Since VS-Code reuses remote server instances, the above is not sufficient to run multiple container images on the same NERSC host at the same time. To get _separate_ (per container image) VS-Code server instances on the _same host_, add something like this to your VS-Code settings _on your local system_:

```
"remote.SSH.serverInstallPath": {
  "someimage~perlmutter": "/global/home/x/xju/.vscode-container/someimage",
  "otherimage~perlmutter": "/global/home/x/xju/.vscode-container/otherimage" 
}
```

### Step 5

Connect to NERSC from with VS-Code _running your local system_:

`F1 > "Connect to Host" > "someimage~perlmutter”` should now start a remote VS-Code session with the VS-Code server component running inside a Shifter container instance at NERSC. The same for `"otherimage~perlmutter"`.

### Tips and tricks

If things don't work, try `"Kill server on remote"` from VS-Code and reconnect.

You can also try starting over from scratch with brute force: Close the VS-Code remote connection. Then, from an external terminal, kill the remote VS-Code server instance (and everything else):

`ssh perlmutter pkill -9 node`

(This will kill _all_ Node.jl processes you own on the remote host.)

Remove the ~/.vscode-server directory in your NERSC home directory.
