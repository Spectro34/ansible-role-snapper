# snapper

This Ansible role installs and configures [Snapper](http://snapper.io), a snapshot tool for Linux filesystems.

The role manages Snapper configurations, snapshots, rollbacks, and hooks for Btrfs and LVM thin-provisioned volumes.

**Original Author:** André Lehmann (aisberg@posteo.de)

This role has been enhanced with additional snapshot management features and is dedicated for SLES and openSUSE systems.

## Supported Distributions

* SLES 16.0 and later
* openSUSE (all versions)

## Requirements

To use Snapper, a compatible filesystem like _Btrfs_ or LVM thin-provisioned volumes needs to be used on the machine. Snapper requires Btrfs or LVM thin-provisioned volumes to create snapshots.

### Collection requirements

No external collections are required.

## Limitations

### Filesystem Requirements

Snapper requires a compatible filesystem (Btrfs or LVM thin-provisioned volumes) to function. The role will fail if attempting to create configurations on unsupported filesystems.

### Snapshot Operations

Snapshot creation, deletion, rollback, and diff operations require an existing Snapper configuration for the target path. Ensure configurations are created before attempting snapshot operations.

### Rollback Operations

Rollback operations may require a system reboot to take effect. The role will not automatically reboot the system.

## Variables

### snapper_templates

Dictionary (key-value pairs) of Snapper templates. The templates can be used when using `create-config` manually. Templates are stored in `/etc/snapper/config-templates/`.

**Default:** `{}`

**Type:** `dict`

Each key in the dictionary represents a template name, and the value is a dictionary of configuration variables. Available configuration variables are documented in the [Available Configuration Variables](#available-configuration-variables) section.

```yaml
snapper_templates:
  default:
    TIMELINE_LIMIT_HOURLY: 2
    TIMELINE_LIMIT_DAILY: 12
    TIMELINE_LIMIT_WEEKLY: 4
  custom:
    TIMELINE_CREATE: false
    NUMBER_LIMIT: 100
```

### snapper_configs

List of Snapper configurations to be present on the system. Each configuration is a dictionary with the following keys:

**Default:** `[]`

**Type:** `list` of `dict`

#### Required keys

* `name` (required): Name of the configuration
* `path` (required): Path to the volume (e.g. Btrfs subvolume)

#### Optional keys

* `vars` (optional): Dictionary of configuration variables. Available variables can be found in the [Available Configuration Variables](#available-configuration-variables) section.

```yaml
snapper_configs:
  - path: /
    name: root
    vars:
      TIMELINE_LIMIT_HOURLY: 5
      TIMELINE_LIMIT_DAILY: 7
      TIMELINE_LIMIT_WEEKLY: 4
      TIMELINE_LIMIT_MONTHLY: 12
  - path: /home
    name: home
    vars:
      TIMELINE_CREATE: true
      TIMELINE_LIMIT_HOURLY: 10
      TIMELINE_LIMIT_DAILY: 30
```

### snapper_configs_absent

List of Snapper configurations to be absent on the system. Each item should be a string containing the configuration name.

**Default:** `[]`

**Type:** `list` of `str`

```yaml
snapper_configs_absent:
  - old_config
  - test_config
```

### snapper_timer_timeline_enabled

Enable or disable the timeline timer. The timeline timer creates hourly snapshots automatically.

**Default:** `true`

**Type:** `bool`

```yaml
snapper_timer_timeline_enabled: true
```

### snapper_timer_cleanup_enabled

Enable or disable the cleanup timer. The cleanup timer removes old snapshots according to the configuration.

**Default:** `true`

**Type:** `bool`

```yaml
snapper_timer_cleanup_enabled: true
```

### snapper_timer_boot_enabled

Enable or disable the boot timer. The boot timer creates a snapshot at boot time.

**Default:** `false`

**Type:** `bool`

```yaml
snapper_timer_boot_enabled: false
```

### snapper_snapshots

List of snapshots to create. Each item is a dictionary with the following optional keys:

**Default:** `[]`

**Type:** `list` of `dict`

#### Optional keys

* `config` (optional): Configuration name (default: uses default configuration)
* `description` (optional): Description for the snapshot
* `cleanup` (optional): Cleanup algorithm (e.g., `number`, `timeline`, `empty-pre-post`)
* `userdata` (optional): User data string
* `read_only` (optional): Create read-only snapshot (default: `false`)
* `type` (optional): Snapshot type (`single`, `pre`, `post`)
* `pre_number` (optional): Pre-snapshot number for post-snapshots

```yaml
snapper_snapshots:
  - config: root
    description: "Manual backup before update"
    cleanup: number
  - config: root
    description: "Pre-installation snapshot"
    type: pre
    userdata: "install=package1,package2"
  - config: root
    description: "Post-installation snapshot"
    type: post
    pre_number: 5
```

### snapper_snapshots_delete

List of snapshots to delete. Each item is a dictionary with the following keys:

**Default:** `[]`

**Type:** `list` of `dict`

#### Required keys

* `number` (required): Snapshot number to delete

#### Optional keys

* `config` (optional): Configuration name (default: uses default configuration)

```yaml
snapper_snapshots_delete:
  - number: 10
    config: root
  - number: 11
```

### snapper_snapshots_list

List of configurations to query snapshots from. Each item is a dictionary with the following optional keys:

**Default:** `undefined` (not set)

**Type:** `list` of `dict`

#### Optional keys

* `config` (optional): Configuration name (default: uses default configuration)
* `type` (optional): Filter by snapshot type (`single`, `pre`, `post`)
* `columns` (optional): List of columns to display

```yaml
snapper_snapshots_list:
  - config: root
    type: single
  - config: home
    columns:
      - number
      - type
      - description
```

### snapper_rollbacks

List of rollback operations. Each item is a dictionary with the following keys:

**Default:** `[]`

**Type:** `list` of `dict`

#### Required keys

* `number` or `snapshot` (required): Snapshot number or identifier to rollback to

#### Optional keys

* `config` (optional): Configuration name (default: uses default configuration)
* `description` (optional): Description for the rollback snapshot
* `cleanup` (optional): Cleanup algorithm

```yaml
snapper_rollbacks:
  - config: root
    number: 5
    description: "Rollback to snapshot 5"
    cleanup: number
```

### snapper_diffs

List of diff operations to show differences between snapshots. Each item is a dictionary with the following keys:

**Default:** `[]`

**Type:** `list` of `dict`

#### Required keys

* `snapshot1` (required): First snapshot number or identifier
* `snapshot2` (required): Second snapshot number or identifier

#### Optional keys

* `config` (optional): Configuration name (default: uses default configuration)
* `files` (optional): List of specific files to compare

```yaml
snapper_diffs:
  - config: root
    snapshot1: 5
    snapshot2: 6
  - config: root
    snapshot1: 10
    snapshot2: 11
    files:
      - /etc/passwd
      - /etc/group
```

### snapper_status

List of status operations to show status between snapshots. Each item is a dictionary with the following keys:

**Default:** `[]`

**Type:** `list` of `dict`

#### Required keys

* `snapshot1` (required): First snapshot number or identifier
* `snapshot2` (required): Second snapshot number or identifier

#### Optional keys

* `config` (optional): Configuration name (default: uses default configuration)
* `files` (optional): List of specific files to check

```yaml
snapper_status:
  - config: root
    snapshot1: 5
    snapshot2: 6
  - config: root
    snapshot1: 10
    snapshot2: 11
    files:
      - /etc
```

### snapper_hooks

List of pre/post snapshot hooks. Each item is a dictionary with the following keys:

**Default:** `[]`

**Type:** `list` of `dict`

#### Required keys

* `config` (required): Configuration name
* `type` (required): Hook type (`pre` or `post`)
* `script` (required): Path to the script

#### Optional keys

* `state` (optional): State of the hook (`present` or `absent`, default: `present`)

```yaml
snapper_hooks:
  - config: root
    type: pre
    script: /usr/local/bin/pre-snapshot.sh
  - config: root
    type: post
    script: /usr/local/bin/post-snapshot.sh
  - config: root
    type: pre
    script: /usr/local/bin/old-hook.sh
    state: absent
```

## Available Configuration Variables

The following variables can be used in `snapper_configs[].vars` or `snapper_templates`:

### Timeline Configuration

* `TIMELINE_CREATE`: Enable/disable timeline snapshots (default: `true`)
* `TIMELINE_CLEANUP`: Enable/disable timeline cleanup (default: `true`)
* `TIMELINE_MIN_AGE`: Minimum age for timeline snapshots in seconds (default: `1800`)
* `TIMELINE_LIMIT_HOURLY`: Maximum number of hourly snapshots (default: `10`)
* `TIMELINE_LIMIT_DAILY`: Maximum number of daily snapshots (default: `10`)
* `TIMELINE_LIMIT_WEEKLY`: Maximum number of weekly snapshots (default: `0`)
* `TIMELINE_LIMIT_MONTHLY`: Maximum number of monthly snapshots (default: `10`)
* `TIMELINE_LIMIT_YEARLY`: Maximum number of yearly snapshots (default: `10`)

### Number Cleanup Configuration

* `NUMBER_CLEANUP`: Enable/disable number cleanup (default: `true`)
* `NUMBER_MIN_AGE`: Minimum age for number cleanup in seconds (default: `1800`)
* `NUMBER_LIMIT`: Maximum number of snapshots (default: `50`)
* `NUMBER_LIMIT_IMPORTANT`: Maximum number of important snapshots (default: `10`)

### Space Management

* `SPACE_LIMIT`: Fraction of filesystem space snapshots may use (default: `0.5`)
* `FREE_LIMIT`: Fraction of filesystem space that should be free (default: `0.2`)
* `QGROUP`: Btrfs qgroup for space-aware cleanup algorithms

### Other Configuration

* `SUBVOLUME`: Subvolume to snapshot (default: path from config)
* `FSTYPE`: Filesystem type (default: `btrfs`)
* `ALLOW_USERS`: Users allowed to work with config
* `ALLOW_GROUPS`: Groups allowed to work with config
* `SYNC_ACL`: Sync users and groups to .snapshots directory (default: `false`)
* `BACKGROUND_COMPARISON`: Start comparing pre/post snapshots in background (default: `true`)
* `EMPTY_PRE_POST_CLEANUP`: Cleanup empty pre-post pairs (default: `true`)
* `EMPTY_PRE_POST_MIN_AGE`: Minimum age for empty pre-post pair cleanup (default: `1800`)
* `PRE_SNAPSHOT_SCRIPT`: Path to pre-snapshot script
* `POST_SNAPSHOT_SCRIPT`: Path to post-snapshot script

## Examples of Options

### Basic Configuration

Create a Snapper configuration for the root filesystem:

```yaml
snapper_configs:
  - path: /
    name: root
    vars:
      TIMELINE_LIMIT_HOURLY: 5
      TIMELINE_LIMIT_DAILY: 7
```

### Enable Timeline Snapshots

Enable automatic hourly snapshots:

```yaml
snapper_timer_timeline_enabled: true
snapper_configs:
  - path: /
    name: root
    vars:
      TIMELINE_CREATE: true
      TIMELINE_LIMIT_HOURLY: 10
      TIMELINE_LIMIT_DAILY: 10
```

### Disable Timeline Snapshots

Disable automatic snapshots:

```yaml
snapper_configs:
  - path: /
    name: root
    vars:
      TIMELINE_CREATE: false
      TIMELINE_CLEANUP: false
```

### Create Manual Snapshot

Create a manual snapshot before system updates:

```yaml
snapper_snapshots:
  - config: root
    description: "Before system update"
    cleanup: number
```

### Create Pre/Post Snapshot Pair

Create a pre/post snapshot pair for package installation:

```yaml
snapper_snapshots:
  - config: root
    description: "Pre-installation"
    type: pre
    userdata: "packages=nginx,apache2"
  - config: root
    description: "Post-installation"
    type: post
    pre_number: 5
```

### List Snapshots

Query and list snapshots:

```yaml
snapper_snapshots_list:
  - config: root
    type: single
```

### Delete Snapshot

Delete a specific snapshot:

```yaml
snapper_snapshots_delete:
  - number: 10
    config: root
```

### Rollback to Snapshot

Rollback to a previous snapshot:

```yaml
snapper_rollbacks:
  - config: root
    number: 5
    description: "Rollback to snapshot 5"
```

### Compare Snapshots

Show differences between two snapshots:

```yaml
snapper_diffs:
  - config: root
    snapshot1: 5
    snapshot2: 6
```

### Configure Hooks

Configure pre and post snapshot hooks:

```yaml
snapper_hooks:
  - config: root
    type: pre
    script: /usr/local/bin/pre-snapshot.sh
  - config: root
    type: post
    script: /usr/local/bin/post-snapshot.sh
```

## Example Playbooks

### Basic Snapper Setup

Install Snapper and create a basic configuration:

```yaml
---
- name: Configure Snapper
  hosts: all
  become: true

  vars:
    snapper_configs:
      - path: /
        name: root
        vars:
          TIMELINE_LIMIT_HOURLY: 5
          TIMELINE_LIMIT_DAILY: 7
          TIMELINE_LIMIT_WEEKLY: 4

  roles:
    - ansible-role-snapper
```

### Advanced Snapshot Management

Create snapshots, manage them, and configure hooks:

```yaml
---
- name: Advanced Snapper Management
  hosts: all
  become: true

  vars:
    snapper_configs:
      - path: /
        name: root

    snapper_snapshots:
      - config: root
        description: "Manual backup before update"
        cleanup: number
      - config: root
        description: "Pre-installation snapshot"
        type: pre
        userdata: "install=package1,package2"

    snapper_snapshots_list:
      - config: root
        type: single

    snapper_rollbacks:
      - config: root
        number: 5
        description: "Rollback to snapshot 5"

    snapper_diffs:
      - config: root
        snapshot1: 5
        snapshot2: 6

    snapper_hooks:
      - config: root
        type: pre
        script: /usr/local/bin/pre-snapshot.sh
      - config: root
        type: post
        script: /usr/local/bin/post-snapshot.sh

  roles:
    - ansible-role-snapper
```

### Multiple Configurations

Configure Snapper for multiple subvolumes:

```yaml
---
- name: Configure Multiple Snapper Configs
  hosts: all
  become: true

  vars:
    snapper_configs:
      - path: /
        name: root
        vars:
          TIMELINE_LIMIT_HOURLY: 5
          TIMELINE_LIMIT_DAILY: 7
      - path: /home
        name: home
        vars:
          TIMELINE_LIMIT_HOURLY: 10
          TIMELINE_LIMIT_DAILY: 30

  roles:
    - ansible-role-snapper
```

### Using Templates

Create and use Snapper templates:

```yaml
---
- name: Configure Snapper with Templates
  hosts: all
  become: true

  vars:
    snapper_templates:
      default:
        TIMELINE_LIMIT_HOURLY: 2
        TIMELINE_LIMIT_DAILY: 12
        TIMELINE_LIMIT_WEEKLY: 4
      custom:
        TIMELINE_CREATE: false
        NUMBER_LIMIT: 100

    snapper_configs:
      - path: /
        name: root
        vars:
          TIMELINE_LIMIT_HOURLY: 5
          TIMELINE_LIMIT_DAILY: 7

  roles:
    - ansible-role-snapper
```

### Remove Configuration

Remove a Snapper configuration:

```yaml
---
- name: Remove Snapper Configuration
  hosts: all
  become: true

  vars:
    snapper_configs_absent:
      - old_config
      - test_config

  roles:
    - ansible-role-snapper
```

## License

MIT

## Author Information

**Original Author:** André Lehmann (aisberg@posteo.de)

This role has been enhanced with additional snapshot management features and is dedicated for SLES 16+ and openSUSE systems.
