+++
title= "Troubleshooting"
date= 2019-10-30T14:55:30-07:00
draft= false
toc= true
+++

## Troubleshooting

### Finding origin keys

On Linux OS:

    ```bash
    # Linux/MacOS
    ls -la /hab/cache/keys
    ls -la $HOME/.hab/cache.keys
    ```
On Windows:

    ```PS
    # Windows (Powershell 5+)
    ls C:\hab\cache\keys
    ```

### Network access / proxy configuration

If the initial install fails, please check that you have outgoing connectivity, and that you can successfully ping the following:

* `raw.githubusercontent.com`
* `bldr.habitat.sh`

If you have outgoing access via a proxy, please ensure that HTTPS_PROXY is set correctly in your environment.

You also will need to have the following _inbound_ port open for your instance:

* Port 80

In the case that you have configured your proxy for the local session while installing but are still receiving connection refusal errors like the one below, you may want to configure your proxy with the `/etc/environment` file or similar.

    ```output
    -- Logs begin at Mon 2019-06-10 09:02:13 PDT. --
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ∵ Missing package for core/hab-launcher
    Jun 10 09:35:15 <TargetMachine> hab[13161]: » Installing core/hab-launcher
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ☁ Determining latest version of core/hab-launcher in the 'stable' channel
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ✗✗✗
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ✗✗✗ Connection refused (os error 111)
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ✗✗✗
    Jun 10 09:35:15 <TargetMachine> systemd[1]: hab-sup.service: Main process exited, code=exited, status=1/FAILURE
    Jun 10 09:35:15 <TargetMachine> hab[13171]: Supervisor not started.
    Jun 10 09:35:15 <TargetMachine> systemd[1]: hab-sup.service: Unit entered failed state.
    Jun 10 09:35:15 <TargetMachine> systemd[1]: hab-sup.service: Failed with result 'exit-code'
    ```

Please work with your enterprise network admin to ensure the appropriate firewall rules are configured for network access.

### Authentication failure when logging in

If you are not able to log in, please double check the settings that you have configured your OAuth application with, as well as the URLs that you have specified in your `bldr.env` file.

### Unable to retrieve OAuth token

You were able to sign in to the authentication provider, but unable to authenticate with Chef Habitat's OAuth token.

Open the `bldr.env` and verify that:

* **APP_URL** ends with "/\"
* **OAUTH_REDIRECT_URL** ends with "/\"
* **OAUTH_CLIENT_ID** is complete and correct
* **OAUTH_CLIENT_SECRET** is complete and correct

Apply changes to the `bldr.env` by running the install script:

    ```bash
    bash ./install.sh
    ```

Restart the Chef Habitat services:

    ```bash
    sudo systemctl restart hab-sup
    ```

### Self-signed cert files do not exist

The latest version of the Chef Habitat Builder on-prem services looks certificates in the `/hab/cache/ssl` directory. Copy your self-signed certificates directory if they are missing. Follow the naming pattern `appname-cert.cert` or `appname-cert.pem`, for example `automate-cert.cert` or `automate-cert.pem`. Do not use `cert.pem`, which is reserved for the Chef Habitat system. Overwriting this file will cause Chef Habitat Builder to fail.

Restart the Chef Habitat services:

    ```bash
    sudo systemctl restart hab-sup
    ```

### Connection refused (os error 111)

If the proxy was configured for the local session during installation, but you are still seeing connection refusal errors, you may want to configure your proxy with the `/etc/environment` file or something similar. Work with your enterprise network admin to ensure the appropriate firewall rules are configured for network access.

    ```output
    -- Logs begin at Mon 2019-06-10 09:02:13 PDT. --
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ∵ Missing package for core/hab-launcher
    Jun 10 09:35:15 <TargetMachine> hab[13161]: » Installing core/hab-launcher
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ☁ Determining latest version of corehab-launcher in the 'stable' channel
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ✗✗✗
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ✗✗✗ Connection refused (os error 111)
    Jun 10 09:35:15 <TargetMachine> hab[13161]: ✗✗✗
    Jun 10 09:35:15 <TargetMachine> systemd[1]: hab-sup.service: Main process exited,code=exited, status=1/FAILURE
    Jun 10 09:35:15 <TargetMachine> hab[13171]: Supervisor not started.
    Jun 10 09:35:15 <TargetMachine> systemd[1]: hab-sup.service: Unit entered failed state.
    Jun 10 09:35:15 <TargetMachine> systemd[1]: hab-sup.service: Failed with result 'exit-code'
    ```

### Error "sorry, too many clients already"

If the hab services don't come up as expected, use `journalctl -fu hab-sup` to check the service logs (also see below for turning on Debug Logging).

If you see a Postgresql error "sorry, too many clients already", you may need to increase the number of configured connections to the database.

In order to do that, run the following:

`echo 'max_connections=200' | hab config apply "builder-datastore.default" $(date +%s)`

Wait for a bit for the datastore service to restart. If the service does not restart on it's own, you can do a 'sudo systemctl restart hab-sup' to restart things.

### Error "Too many open files"

If you see this error message in the supervisor logs, that may indicate that you need to increase the file ulimit on your system. The Chef Habitat Builder on-prem systemd configuration includes an expanded file limit, however some distributions (eg, on CentOS 7) may require additional system configuration.

For example, add the following to the end of your `/etc/security/limits.conf` file, and restart your system.

    ```text
    * soft nofile 65535
    * hard nofile 65535
    ```

### Error "Text file busy"

Occasionally you may get an error saying "Text file too busy" during install.
If you get this, please re-try the install step again.

### Error when bootstrapping core packages

You may see the following error when bootstrapping the core packages using the script above. If this happens, the bootstrap process will continue re-trying, and the upload will eventually succeed. Be patient and let the process continue until successful completion.

    ```output
    ✗✗✗
    ✗✗✗ Pooled stream disconnected
    ✗✗✗
    ```

If some packages do not upload, you may try re-uploading them manually via the `hab pkg upload` command.

This may also be an indication that your installation may not have sufficient CPU, RAM or other resources, and you may want to either allocate additional resources (eg, if on a VM) or move to a more scaled-up instance.

### Error uploading large packages

By default, the installed services configuration will set a 2GB limit for packages that can be uploaded to the Chef Habitat Builder on-prem. If you need to change the limit, you can do so by injecting an updated config to the Chef Habitat Builder on-prem services.

For example, to change the limit to 3GB, you could do the following:

Create a file called `config.toml` with the following content:

    ```toml
    [nginx]
    max_body_size = "3072m"
    proxy_send_timeout = 360
    proxy_read_timeout = 360

    [http]
    keepalive_timeout = "360s"
    ```

Then, issue the following command:

    ```bash
    hab config apply builder-api-proxy.default $(date +%s) config.toml
    ```

After the config is successfully applied, re-try the upload.

If you have any issues, you may also need to adjust the timeout configuration on the Chef Habitat client.
You can do that via an environment variable: `HAB_CLIENT_SOCKET_TIMEOUT`. The value of this environment variable is a timeout in seconds. So for example, you could do something like this when uploading a file:

    ```bash
    HAB_CLIENT_SOCKET_TIMEOUT=360 hab pkg upload -u http://localhost -z <your auth token> <file>
    ```

### Package shows up in the UI and `hab pkg search`, but `hab pkg install` fails

If you run into a situation where you have a package populated in the Chef Habitat Builder on-prem, but it is failing to install with a `Not Found` status, it is possible that there was a prior problem with populating the Minio backend with the package artifact.

If you have the package artifact on-disk (for example, in the `hab/cache/artifacts` directory), you can try to upload the missing package again with the following command (update the parameters as appropriate):

    ```bash
    hab pkg upload -u http://localhost -z <your auth token> --force <package hart file>
    ```

Note: the --force option above is only available in versions of the `hab` client greater than 0.59.

### on-prem-archive.sh Fails during `populate-depot` with `403` error during core package uploads

When populating your Chef Habitat Builder on-prem with upstream core packages, you may run into an error that looks like this:

    ```output
    Uploading hart files.

    [1/958] Uploading ./core-img-0.5.4-20190201011741-x86_64-linux.hart to the depot at https://your.awesome.depot
      75 B / 75 B | [=========================================] 100.00 % 384 B/s
    ✗✗✗
    ✗✗✗ [403 Forbidden]
    ✗✗✗
    ```

And repeats for every package. Check to make sure you've created the `core` origin and then try again, if you haven't, then the upload will fail.

## Logs

## Logging

### Logging Levels

The recognized values for logging are: `error`, `warn`, `info`, `debug`, and `trace`.
For a more detailed explanation of logging in Chef Habitat, see the [Supervisor Log Configuration Reference](https://www.habitat.sh/docs/reference/#supervisor-log-configuration-reference) and the [Supervisor Log Key](https://www.habitat.sh/docs/reference/#supervisor-log-key) documentation.

### Basic Logging

To turn on and examine the services debug logging in your Habitat installation:

1. Edit the `sudo /hab/svc/builder-api/user.toml` file
1. On the first line, change the log_level from **error** to **debug**

    ```toml
    log_level="debug,tokio_core=error,tokio_reactor=error,zmq=error,hyper=error"
    ```

1. Save and close the file
1. Restart Habitat with `sudo systemctl restart hab-sup` to restart the habitat.
1. Use `journalctl -fu hab-sup` to view the logs.
1. Reset `/hab/svc/builder-api/user.toml` file to the default `log_level=error` and restart the services with `sudo systemctl restart hab-sup` to restore error-level logging.

### RUST_LOG

1. Use `RUST_LOG=debug RUST_BACKTRACE=1` to see an individual command's debug and backtrace.

    ```bash
    # Linux/MacOS
    # replace "hab sup run" with your command
    RUST_LOG=debug RUST_BACKTRACE=1 hab sup run
    ```

1. Edit the `sudo /hab/svc/builder-api/user.toml` file
1. On the second line, change:

    ```toml
    RUST_LOG=debug RUST_BACKTRACE=1
    ```

### Log Rotation

The `builder-api-proxy` service will log (via Nginx) all access and errors to log files in your service directory. Since these files may get large, you may want to add a log rotation script. Below is a sample logrotate file that you can use as an example for your needs:

    ```bash
    /hab/svc/builder-api-proxy/logs/host.access.log
    /hab/svc/builder-api-proxy/logs/host.error.log
    {
            rotate 7
            daily
            missingok
            notifempty
            delaycompress
            compress
            postrotate
                    /bin/kill -USR1 `cat /hab/svc/builder-api-proxy/var/pid 2>/dev/null` 2>/dev/null || true
            endscript
    }
    ```

## License

Copyright (c) 2018 Chef Software Inc. and/or applicable contributors

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0) at `http://www.apache.org/licenses/LICENSE-2.0)`
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## Support

You can also post any questions or issues on the [Habitat Forum](https://forums.habitat.sh/), on our [Slack channel](https://habitat-sh.slack.com), or file issues directly at the [Github repo](https://github.com/habitat-sh/on-prem-builder/issues).
