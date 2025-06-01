build_kali_desktop
=========

Deploy a customizable desktop environment for [Kali Linux](https://www.kali.org/get-kali/).

This role is meant to manage all things specific to Kali without getting in the way of more common system settings and services. Things such as DNS, logging, additional hardening, HIDS/IPS can (and should) be applied using other roles. For example none of these changes prevent you from telling Kali to use unbound or stubby for DNS over TLS to the public web, or even deploying an EDR agent along with auditd for log and event colleciton.

> [!TIP]
> You can also run this role repeatedly without generating the same amount of traffic each time. The role was designed with numerous checks in place, to ensure only missing or updated files, packages, and data are ever retrieved. Nothing is unnecessarily downloaded.

- Select which tool sets you want by specifying the variables noted in `defaults/main.yml`, in your inventory file.
- Handles tools and packages by using a template task file, choose and manage the tools in `vars/main.yml` ([see below](#role-variables))
  - Tasks pull from sources such as apt, pip, GitHub repos, GitHub release files, and individual URLs
- Enables apparmor (including for Firefox)
- Installs a custom Firefox policy JSON file
- Automatically exports and installs the Burpsuite and ZAProxy CA certifactes (learned from [IppSec/parrot-build](https://github.com/IppSec/parrot-build/blob/master/roles/customize-browser/files/getburpcert.sh))
- Enables UFW, allows SSH
- Refreshes the SSH host keys (useful if Kali was pre-installed)
- Adjusts proxychains timeout values for scanning
- Installs [GEF (GDB Enhanced Features)](https://github.com/hugsy/gef)

Requirements
------------

Just Kali Linux. Both real hardware and virtualized environments will work.

Role Variables
--------------

Use `defaults/main.yml` to select tool sets to install. "core" is always present as the base set of packages for general usage.

***Currently only `core`, `pentest`, `webapp`, and `sliver` are ready to use.***

**C2**

- `c2_choice: "none"` Set to `sliver` to install and configure the sliver C2 framework
- `tools-c2.yml` determines what C2 tasks to include based on the value of `c2_choice`
- More C2 frameworks can be added as their own task file, for example: `tools-c2-covenant.yml`

**Tools**

- `core_tools: true` Set to `false` if you don't want the core tools
- `pentest_tools: false` Set to `true` to iunclude pentest tools
- `webapp_tools: false` Set to `true` to iunclude webapp tools
- `wireless_tools: false` Set to `true` to include wireless tools
- `analysis_tools: false` Set to `true` to include analysis tools
- `forensics_tools: false` Set to `true` to include forensics tools
- `development_tools: false` Set to `true` to include development tools

All of the tools and packages are maintained through `vars/main.yml`, and each `tools-*` task file shares the same structure to maintain its own tool set. Each of these sources can be used to obtain and install tools:

- apt packages
- pip / go / ruby / npm packages
- git release files
- git repos to clone
- any other individual files to download
- any additional changes or adjustments specific to this task file

For example, the `tasks/tools-TEMPLATE.yml` was used to create a `tools-forensics.yml` task file.

`apt_list_REPLACE_THIS` would change to `apt_list_forensics`, and `vars/main.yml` would contain the `apt_list_forensics` variable of apt packages related to forensics.

Populate the remaining areas with any necessary lists; `pip_list_forensics`, `author_repo_list_forensics`, and so on.

Dependencies
------------

None.

Example Playbook
----------------

Using an inventory.yml file, referencing a vault for the sudo password:

```yml
remotegroup:
  hosts:
    10.10.0.41:
      ansible_port: 2222
      ansible_user: kali
      webapp_tools: true
      c2_choice: "sliver"
      ansible_become_password: "{{ kali_sudo_pass }}"
      ansible_become_method: sudo
vars:
    #per_group_var: "some_other_value"
    #per_group_list:
      #- list_item_1
      #- list_item_2

```

You could use this playbook file:

```yml
- name: "Default Playbook"
  hosts:
    - remotegroup
  roles:
    - role: "build_kali_desktop"
```

To execute it on a remote host:

```bash
echo "Enter Vault Password"; read -r -s vault_pass; export ANSIBLE_VAULT_PASSWORD=$vault_pass
# [type vault password here, then enter]
ansible-playbook -i <inventory> -e "@~/vault.yml" --vault-pass-file <(cat <<<$ANSIBLE_VAULT_PASSWORD) -v ./playbook.yml
```

Alternatively, execute locally with (keep in mind [non-string values require JSON format](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#key-value-format)):

```bash
ansible-playbook -i "localhost," -c local -e '{"webapp_tools":true,"c2_choice":"sliver"}' --ask-become-pass -v ./playbook.yml
```

License
-------

[MIT](https://github.com/straysheep-dev/ansible-configs/blob/main/LICENSE): All files unless otherwise noted are released under the MIT license.

[BSD-3-Clause](https://github.com/IppSec/parrot-build/tree/master/roles/customize-browser#license): The method for obtaining the Burpsuite CA was learned and taken from IppSec's parrot-build [getburpcert.sh](https://github.com/IppSec/parrot-build/blob/master/roles/customize-browser/files/getburpcert.sh). That task block here will remain under the BSD-3-Clause.

[MPL-2.0](https://github.com/straysheep-dev/ansible-role-build_kali_desktop/blob/main/LICENSES/LICENSE-MPL-2.0), [GPL-2.0-or-later](https://gitlab.com/apparmor/apparmor/-/raw/master/LICENSE): The apparmor profiles here are [my own fork](https://github.com/straysheep-dev/linux-configs/tree/main/apparmor) of the original file(s) that used to ship with Ubuntu before Firefox moved to snap being the default. These work with both, Firefox and Firefox-ESR, with local overrides to protect `$HOME` and other paths via [usr.bin.firefox.local](https://github.com/straysheep-dev/linux-configs/blob/main/apparmor/apparmor-usr.bin.firefox.local). The `MPL-2.0` file and the original `usr.bin.firefox.apparmor.14.10` profile are from the [Firefox source on launchpad](https://bazaar.launchpad.net/~mozillateam/firefox/firefox.focal/files/head:/debian). That repo also contains a [copyright](https://github.com/straysheep-dev/ansible-role-build_kali_desktop/blob/LICENSES/copyright) file which listed `usr.bin.firefox.apparmor.14.10` as falling under the MPL-2.0.

- `apparmor-usr.bin.firefox` falls under the `MPL-2.0` license
- `apparmor-firefox.abstractions` appears to be created by `aa-update-browser` and falls under the `GPL-2.0-or-later` license
- `apparmor-usr.bin.firefox.local` is my local override file, and falls under the default `MIT` license of this repo

Author Information
------------------

[straysheep-dev/ansible-configs](https://github.com/straysheep-dev/ansible-configs/)
