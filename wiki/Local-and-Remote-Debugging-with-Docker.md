## Docker Examples  

Some examples below are in lua, if you want to set up `nvim-dap` in a `.vim` file, you have to wrap the code blocks like this:

```
lua <<EOF
-- lua code goes here
EOF
```

See `:help lua-commands`

Otherwise you can add a `lua/` dir inside the dir that hosts your `init.vim` and configure all of this in raw lua like this:

```sh
cd ~/.config/nvim
mkdir lua
cd lua
touch debuggers.lua
```
then you can require this lua file inside your `init.vim` like this:
```vim
lua require"debuggers"
```
### Contents

- [Shared](#Shared)
- [Go](#Go)
- [Python](#Python)

## Shared

There are some basic steps that remain the same across both remote and local Docker Debugging:
1. Expose a port in your Docker container responsible for debugging.
2. Modify your containers [security settings](#Docker-Security-Settings) to allow ptrace to run correctly.
3. Install the program/package responsible for starting the debug server (and maybe adapter) in your container.
4. Install the program/package responsible for launching the debug adapter locally (if necessary).
5. Start/Attach the debug server (and potentially adapter) inside your container to your code.
6. Launch the debug adapter locally (if not done by the debug server in the container).
7. Connect nvim-dap to either the debug adapter you spawned locally or the adapter spun up by your debug server in the container.

For remote debugging you are probably best off creating an SSH tunnel between your remote Docker instance and your local machine for the port you designate for debugging:
Running something like:
```sh
ssh -L localhost:${your_debug_port}:${remote_docker_host}:${your_debug_port} ${remote_docker_host} tail -f /dev/null
```
from inside your container should do the trick.

### Docker Security Settings
For debugging in Docker (both remotely and locally) you will need to allow a couple of security settings see [here](https://stackoverflow.com/questions/19215177/how-to-solve-ptrace-operation-not-permitted-when-trying-to-attach-gdb-to-a-pro) for more information.
These settings can either be achieved:
- through the command line as `docker run --cap-add=SYS_PTRACE --security-opt seccomp=unconfined`
- in a Docker compose file via:
```docker
security_opt:
  - seccomp:unconfined
cap_add:
  - SYS_PTRACE
```

## Go
- Install [vscode-go](https://github.com/golang/vscode-go). This lets you spin up a debug adapter locally and is not necessary inside the Docker container.
  - `git clone https://github.com/golang/vscode-go`
  - `cd vscode-go`
  - `npm run compile`
- Install [delve](https://github.com/go-delve/delve/tree/master/Documentation/installation) on both your local machine and the Docker container.
  - `go get github.com/go-delve/delve/cmd/dlv`
  - or via package manager (`pacman -S delve`)
  - or in Docker `RUN go get github.com/go-delve/delve/cmd/dlv`
- Attach delve to your code in the container:
```sh
dlv --listen :${your_debug_port} --headless --accept-multiclient --api-version 2 attach $(pgrep -fn ${the_service_name})
```
This command could be replaced by:
```sh
dlv --listen :${your_debug_port} --headless --accept-multiclient --api-version 2 exec ${your_compiled_go_program}
```
If your container does not start the go process itself already.
- Launch the debug adapter locally via nvim-dap:
```lua
local M = {}
M.launch_go_debugger = function(args)
    local dap = require "dap"
    -- The adapter has not been started yet.
    -- Spin it up.
    goLaunchAdapter = {
        type = "executable";
        command = "node";
        args = {os.getenv("HOME") .. "/vscode-go/dist/debugAdapter.js"};
    }

    goLaunchConfig = {
        type = "go";
        request = "attach";
        mode = "remote";
        name = "Remote Attached Debugger";
        dlvToolPath = os.getenv('HOME') .. "/go/bin/dlv";  -- Or wherever your local delve lives.
        remotePath = ${where_your_local_copy_of_the_code_in_your_container_lives};
        port = ${your_exposed_container_port};
        cwd = vim.fn.getcwd();
   }
   -- If you want you can even have nvim be responsible for the `delve` launch step above:
   --  vim.fn.system({"${some_script_that_starts_dlv_in_your_container}", ${script_args})
   local session = dap.launch(goLaunchAdapter, goLaunchConfig);
    if session == nil then
        io.write("Error launching adapter");
    end
    dap.repl.open()
end
```  

## Python

Python Docker debugging is a little simpler since the Python debug package [debugpy](https://github.com/microsoft/debugpy) can run both the server and adapter inside your container.

- Install Debugpy both locally and in your Docker container:
  - This does not need to be in a separate venv if there is a different one you would prefer to install to feel free.
```sh
python -m venv path/to/virtualenvs/debugpy
path/to/virtualenvs/debugpy/bin/python -m pip install debugpy
```
- Attach a debugpy server to your code in the container and launch a debug adapter.
```sh
path/to/virtualenvs/debugpy/bin/python -m debugpy --listen 0.0.0.0:${your_debug_port} --pid $(pgrep -nf ${your_running_program})
```
See [here](https://github.com/microsoft/debugpy/wiki/Command-Line-Reference#quick-and-easy) for some more examples.
Unlike with our Go example the `debugpy` command will also take care of launching our debug adapter so our nvim-dap code needs to be an attach configuration, not a launch one.
- Attach nvim-dap to the running debugpy DAP adapter
```lua
local M = {}
M.attach_python_debugger = function(args)
    local dap = require "dap"
    local host = args[1] -- This should be configured for remote debugging if your SSH tunnel is setup.
    -- You can even make nvim responsible for starting the debugpy server/adapter:
    --  vim.fn.system({"${some_script_that_starts_debugpy_in_your_container}", ${script_args}})
    pythonAttachAdapter = {
        type = "server";
        host = host;
        port = tonumber(${your_debug_port});
    }
    pythonAttachConfig = {
        type = "python";
        request = "attach";
        connect = {
            port = tonumber(${your_debug_port});
            host = host;
        };
        mode = "remote";
        name = "Remote Attached Debugger";
        cwd = vim.fn.getcwd();
        pathMappings = {
            {
                localRoot = vim.fn.getcwd(); -- Wherever your Python code lives locally.
                remoteRoot = "/usr/src/app"; -- Wherever your Python code lives in the container.
            };
        };
    }
    local session = dap.attach(host, tonumber(${your_debug_port}), pythonAttachConfig)
    if session == nil then
        io.write("Error launching adapter");
    end
    dap.repl.open()
end
```