<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up LibreSpeed

This is an [Ansible](https://www.ansible.com/) role which installs [LibreSpeed](https://github.com/librespeed/speedtest) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

LibreSpeed is a self-hosted speed test, which runs in the browser and measures the connection between the client and your server.

See the project's [documentation](https://github.com/librespeed/speedtest/blob/master/README.md) to learn what LibreSpeed does and why it might be useful to you.

## Adjusting the playbook configuration

To enable LibreSpeed with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# librespeed                                                           #
#                                                                      #
########################################################################

librespeed_enabled: true

########################################################################
#                                                                      #
# /librespeed                                                          #
#                                                                      #
########################################################################
```

### Set the hostname

To enable LibreSpeed you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
librespeed_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Set the title (optional)

You can set the title of the speed test and the slogan shown below it (the slogan only in [the new design](#use-the-new-design-optional)) by adding the following configuration to your `vars.yml` file.

```yaml
librespeed_environment_variables_title: Example Speed Test
librespeed_environment_variables_tagline: Test your connection to example.com
```

### Set a subpath (optional)

It is possible to serve the instance under a subpath by adding the following configuration to your `vars.yml` file.

```yaml
librespeed_path_prefix: /librespeed
```

### Use the new design (optional)

LibreSpeed comes with a classic and a new design, and uses the classic one by default. To use the new design, add the following configuration to your `vars.yml` file:

```yaml
librespeed_environment_variables_use_new_design: true
```

>[!NOTE]
> The privacy policy of the classic design says that telemetry is enabled and that test results are stored, even if [saving them](#save-test-results-optional) is not enabled. The new design does not show one unless it is.

### Configure the test (optional)

There are variables for common test settings, such as the durations in seconds:

```yaml
librespeed_config_time_dl_max: 15
librespeed_config_time_ul_max: 15
```

LibreSpeed multiplies the measured speeds by 1.06 to account for the overhead of network protocols, which the browser cannot see. To have it report the speed at which data is actually transferred, add the following configuration to your `vars.yml` file:

```yaml
librespeed_config_overheadCompensationFactor: 1
```

>[!NOTE]
> These settings apply to [the new design](#use-the-new-design-optional). The classic design has its own ones.

### Customize the appearance (optional)

You can replace files of the web interface by mounting your own over them. For example, to change the logo and the colors of [the new design](#use-the-new-design-optional), add the following configuration to your `vars.yml` file:

```yaml
librespeed_container_additional_volumes_custom:
  - type: bind
    src: /path/to/logo.svg
    dst: /speedtest/frontend/images/logo.svg
    options: readonly
  - type: bind
    src: /path/to/colors.css
    dst: /speedtest/frontend/styling/colors.css
    options: readonly
```

The files are copied when the container starts, so restart the service after changing them. See [LibreSpeed's `frontend` directory](https://github.com/librespeed/speedtest/tree/master/frontend) for the files which can be replaced.

### Show details about the client's ISP (optional)

By default LibreSpeed shows the client's ISP and country, looking them up in a database which comes with the container image. To hide them in [the new design](#use-the-new-design-optional), add the following configuration to your `vars.yml` file:

```yaml
librespeed_config_getIp_ispInfo: false
```

To look them up on [ipinfo.io](https://ipinfo.io) and also show the distance from the server, set an API key:

```yaml
librespeed_environment_variables_ipinfo_apikey: YOUR_API_KEY_HERE
```

>[!WARNING]
> With an API key, clients' IP addresses are sent to ipinfo.io. This can be requested by anyone who can reach the instance, even if `librespeed_config_getIp_ispInfo` is `false`. LibreSpeed's privacy policy does not mention it.

### Save test results (optional)

LibreSpeed can save test results in a SQLite database, let clients share them, and list them on a password-protected page. To enable it, add the following configuration to your `vars.yml` file:

```yaml
librespeed_environment_variables_telemetry: true

# The password of the page which lists the results. Only letters, digits, `_`, `.` and `-` are supported.
librespeed_environment_variables_password: YOUR_PASSWORD_HERE

# The email address which the privacy policy shows for requests to delete data
librespeed_environment_variables_gdpr_email: privacy@example.com
```

The page is available at `/results/stats.php`.

>[!NOTE]
> The results include clients' IP addresses, unless `librespeed_environment_variables_redact_ip_addresses` is set to `true`. The redaction is not complete: parts of IPv6 addresses may remain, and LibreSpeed's `/results/sanitycheck.php` page saves a test entry with the IP address of anyone who opens it.

Anyone who knows the ID of a result can view it. To not number the results sequentially, you can have their IDs obfuscated (this is not a strong protection):

```yaml
librespeed_environment_variables_enable_id_obfuscation: true

# The value can be generated with `echo "0x$(openssl rand -hex 4)"` or in another way. Make sure to keep the quotes.
librespeed_environment_variables_obfuscation_salt: "YOUR_SALT_HERE"
```

### Use several test servers (optional)

Other LibreSpeed servers can be offered to clients besides the instance itself. To do so, add the following configuration to your `vars.yml` file:

```yaml
librespeed_environment_variables_mode: dual

librespeed_server_list_custom:
  - name: Example
    server: //speedtest.example.com/
    dlURL: garbage.php
    ulURL: empty.php
    pingURL: empty.php
    getIpURL: getIP.php
```

Use the `frontend` mode to offer only the other servers, or the `backend` mode to have the instance serve as a test server without a web interface.

### Avoiding inaccurate results

- A test occupies a large part of the server's bandwidth, so the results of clients testing at the same time lower each other.
- A compression middleware or a rate limit on the reverse proxy would affect the results.
- HTTP/2 and HTTP/3 need more CPU time than HTTP/1.1, so slow clients or servers may not reach the full speed of a fast connection with them.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `librespeed_environment_variables_additional_variables` variable
- [`templates/settings.json.j2`](../templates/settings.json.j2) for the test settings of [the new design](#use-the-new-design-optional). You can override settings (even those that don't have dedicated playbook variables) using the `librespeed_configuration_extension_json` variable

Refer to [this page on the official documentation](https://github.com/librespeed/speedtest/blob/master/doc_docker.md) for LibreSpeed's environment variables, and [this section](https://github.com/librespeed/speedtest/blob/master/doc.md#test-parameters) for a complete list of its test settings.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, LibreSpeed becomes available at the specified hostname like `https://example.com`. To use it, open the URL on the browser and start the test.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu librespeed` (or how you/your playbook named the service, e.g. `mash-librespeed`).
