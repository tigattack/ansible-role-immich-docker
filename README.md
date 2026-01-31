# Ansible Role: immich_docker

[![Build Status](https://img.shields.io/github/actions/workflow/status/tigattack/ansible-role-immich-docker/test.yml?branch=main&label=Lint%20%26%20Test)](https://github.com/tigattack/ansible-role-immich-docker/actions?query=workflow:Test)
[![Ansible Galaxy](https://img.shields.io/ansible/role/d/tigattack/immich_docker)](https://galaxy.ansible.com/ui/standalone/roles/tigattack/immich_docker/)

Ansible role to deploy [Immich](https://immich.app/) ([GitHub](https://github.com/immich-app/immich)) in Docker.

Install the role: `ansible-galaxy role install tigattack.immich_docker`

See [Example Playbooks](#example-playbooks) below.

## Prerequisites

* Docker. I recommend the [geerlingguy.docker](https://github.com/geerlingguy/ansible-role-docker) role.
* [community.docker](https://galaxy.ansible.com/ui/repo/published/community/docker/) Ansible collection. See [requirements.yml](requirements.yml).

## Role Variables

> [!TIP]
> Once installed, you can run `ansible-doc -t role tigattack.immich_docker` to see role documentation. You can also see more in [defaults/main.yml](defaults/main.yml).

<!-- BEGIN ANSIBLE ROLE VARIABLES -->

### Global Settings

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_container_base_name` | string | `immich` | ❌ |
| `immich_docker_containers_state` | string | `healthy` | ❌ |
| `immich_docker_container_network` | string | `immich` | ❌ |

**Details:**
- `immich_docker_container_base_name`: Base name for all Immich containers. Individual containers will be suffixed (e.g., `immich-server`, `immich-postgres`).
- `immich_docker_containers_state`: The desired state of the Immich containers. For valid options, see the [`state` parameter for the `community.docker.docker_container` module](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html#parameter-state).
- `immich_docker_container_network`: Name of the Docker network to connect the Immich containers to. Note that `bridge`, the default Docker container network, must NOT be used.

> [!NOTE]
> `community.docker` versions >=4.7.0 can use the `healthy` state, whereas <4.7.0 need to use `started`. See <https://github.com/ansible-collections/community.docker/issues/890>.

### Core Application Container Settings

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_version` | string | `release` | ❌ |
| `immich_docker_env_vars` | dict | `{}` | ❌ |
| `immich_docker_transcoding_acceleration` | string | `cpu` | ❌ |

**Details:**
- `immich_docker_version`: Immich server Docker image version (tag). Can be `release` or any other valid image tag (e.g. `v2.1.0`, etc). See more at <https://github.com/immich-app/immich/pkgs/container/immich-server>
- `immich_docker_env_vars`: Environment variables to pass to the Immich server container. Empty dictionary (no env vars) by default.
- `immich_docker_transcoding_acceleration`: Hardware acceleration backend for video transcoding.

#### Hardware Transcoding

This role supports configuration for hardware transcoding to reduce CPU load during video processing. Hardware transcoding produces larger files than software transcoding but significantly reduces transcoding time and CPU usage.

**Configuration:**

```yaml
# NVIDIA NVENC example
immich_docker_transcoding_acceleration: nvenc

# Intel Quick Sync example
immich_docker_transcoding_acceleration: quicksync

# More available - See docs link below.
```

> [!IMPORTANT]
> - NVENC requires NVIDIA Container Toolkit on the host
> - After enabling hardware acceleration, configure the transcoding settings in the Immich Admin panel
> - Hardware transcoding is separate from ML hardware acceleration (below) - you can use different backends for each

For detailed configuration, prerequisites, and troubleshooting information, see: <https://docs.immich.app/features/hardware-transcoding/>

### Storage Settings

For each data volume, you may choose between a bind mount and a Docker volume proper.

These options are mutually exclusive. Using the `immich_docker_data_*` variables as an example, the following two scenarios explain:

- `immich_docker_data_use_docker_volume=false`
  - Value of `immich_docker_data_bind_path` is used
  - Value of `immich_docker_data_volume` is ignored regardless of value
- `immich_docker_data_use_docker_volume=true`
  - Value of `immich_docker_data_volume` is used
  - Value of `immich_docker_data_bind_path` is ignored regardless of value

#### Bind Mounts

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_bind_paths_root` | path | `/opt/immich` | ❌ |
| `immich_docker_data_bind_path` | path | `<immich_docker_bind_paths_root>/data` | ❌ |
| `immich_docker_model_cache_bind_path` | path | `<immich_docker_bind_paths_root>/model_cache` | ❌ |
| `immich_docker_db_data_bind_path` | path | `<immich_docker_bind_paths_root>/db` | ❌ |

**Details:**
- `immich_docker_bind_paths_root`: Root path for bind mounts. Used by the following variables' default values. If all are overridden with alternative paths, this has no effect.
- `immich_docker_data_bind_path`: Immich data path on the host.
- `immich_docker_model_cache_bind_path`: Immich machine learning model cache path on the host.
- `immich_docker_db_data_bind_path`: PostgreSQL data path on the host.

#### Docker Volumes

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_data_use_docker_volume` | bool | `false` | ❌ |
| `immich_docker_data_volume` | dict | See below | ❌ |
| `immich_docker_model_cache_use_docker_volume` | bool | `false` | ❌ |
| `immich_docker_model_cache_volume` | dict | See below | ❌ |
| `immich_docker_db_data_use_docker_volume` | bool | `false` | ❌ |
| `immich_docker_db_data_volume` | dict | See below | ❌ |

**Defaults:** All Docker volumes default to local volumes, named `immich_<type>` (where `type` is `data`, `db`, etc.), with no driver options specified.

**Details:**
- `immich_docker_data_use_docker_volume`: Use a Docker volume for Immich data instead of a bind mount.
- `immich_docker_data_volume`: Dictionary of settings for the Immich data Docker volume. See <https://docs.ansible.com/ansible/latest/collections/community/docker/docker_volume_module.html>.
- `immich_docker_model_cache_use_docker_volume`: Use a Docker volume for Immich machine learning model cache instead of a bind mount.
- `immich_docker_model_cache_volume`: Dictionary of settings for the Immich model cache Docker volume. See <https://docs.ansible.com/ansible/latest/collections/community/docker/docker_volume_module.html>.
- `immich_docker_db_data_use_docker_volume`: Use a Docker volume for PostgreSQL data instead of a bind mount.
- `immich_docker_db_data_volume`: Dictionary of settings for the PostgreSQL data Docker volume. See <https://docs.ansible.com/ansible/latest/collections/community/docker/docker_volume_module.html>.

**Example data volume with CIFS:**

```yaml
immich_docker_data_volume:
  name: immich_data
  driver: local
  driver_options:
    type: cifs
    device: //nas/data/photos/immich
    o: addr=nas,username=immich,password=hunter2,dir_mode=0770,file_mode=0660
```

#### Volume Change Behavior

> [!IMPORTANT]
> When a Docker volume configuration changes (e.g., changing driver options for a CIFS mount), **all Immich containers will be stopped and not restarted**. This prevents dependency issues (e.g., app containers trying to connect to a stopped database) and ensures a clean state for data migration.

**What happens during a volume change:**
1. The affected container is removed and the new volume is created
2. **All Immich containers are stopped** (server, ML, Redis, PostgreSQL)
3. A notice is displayed indicating which volumes changed and manual migration is required
4. All containers remain stopped until the role is run again after migration

**Migration workflow:**
```bash
# 1. Run the role - volume change detected, all containers stopped
ansible-playbook playbook.yml

# 2. Manually migrate data from old volume(s) to new volume(s)
# (Use docker cp, rsync, or mount volumes directly)

# 3. Run the role again to start all containers
ansible-playbook playbook.yml
```

#### Extra Mounts

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_extra_volumes` | list | `[]` | ❌ |
| `immich_docker_machine_learning_extra_volumes` | list | `[]` | ❌ |

**Details:**
- `immich_docker_extra_volumes`: A list of extra bind mounts for the Immich server container. See the [`mounts` parameter of the `community.docker.docker_container` module](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html#parameter-mounts).
- `immich_docker_machine_learning_extra_volumes`: A list of extra bind mounts for the Immich machine learning container. See the [`mounts` parameter of the `community.docker.docker_container` module](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html#parameter-mounts).

### Machine Learning Settings

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_machine_learning_version` | string | `release` | ❌ |
| `immich_docker_machine_learning_acceleration` | string | `cpu` | ❌ |
| `immich_docker_machine_learning_env_vars` | dict | `{}` | ❌ |

**Details:**
- `immich_docker_machine_learning_version`: Immich machine learning Docker image version (tag).
- `immich_docker_machine_learning_acceleration`: Hardware acceleration backend for machine learning. See <https://docs.immich.app/features/ml-hardware-acceleration/> for more information.
- `immich_docker_machine_learning_env_vars`: Additional environment variables to pass to the Immich machine learning container.

> [!IMPORTANT]
> For CUDA acceleration, ensure you have the NVIDIA Container Toolkit installed and your GPU has compute capability 5.2 or greater. See <https://docs.immich.app/features/ml-hardware-acceleration/> for more details.

> [!NOTE]
> When using hardware acceleration (i.e., `immich_docker_machine_learning_acceleration` is not `cpu`), the ML container image will automatically use the appropriate tag suffix (e.g., `-cuda`, `-armnn`).

#### Hardware Acceleration

This role supports configuration of Immich's machine learning service.

**Configuration Examples:**

```yaml
# NVIDIA CUDA example
immich_docker_machine_learning_acceleration: cuda

# ARM NN example
immich_docker_machine_learning_acceleration: armnn

# Rockchip NPU example
immich_docker_machine_learning_acceleration: rknn

# Multi-GPU Support (CUDA/OpenVINO)
immich_docker_machine_learning_acceleration: cuda
immich_docker_machine_learning_env_vars:
  MACHINE_LEARNING_DEVICE_IDS: "0,1"  # Use GPUs 0 and 1
  MACHINE_LEARNING_WORKERS: "2"       # One worker per GPU
```

For detailed information, prerequisites, and troubleshooting, see the official documentation: <https://docs.immich.app/features/ml-hardware-acceleration/>

### PostgreSQL Settings

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_deploy_postgres` | bool | `true` | ❌ |
| `immich_docker_postgres_version` | string | `14-vectorchord0.4.3-pgvectors0.2.0` | ❌ |
| `immich_docker_postgres_image` | string | `ghcr.io/immich-app/postgres:{{ immich_docker_postgres_version }}` | ❌ |
| `immich_docker_db_username` | string | `postgres` | ❌ |
| `immich_docker_db_database_name` | string | `immich` | ❌ |
| `immich_docker_db_password` | string | - | ✅ |

**Details:**
- `immich_docker_deploy_postgres`: Deploy a PostgreSQL instance for Immich. Set to `false` if using an external PostgreSQL instance.
- `immich_docker_postgres_version`: PostgreSQL Docker image version for Immich.
- `immich_docker_postgres_image`: PostgreSQL Docker image for Immich.
- `immich_docker_db_username`: PostgreSQL username for Immich.
- `immich_docker_db_database_name`: PostgreSQL database name for Immich.
- `immich_docker_db_password`: PostgreSQL password for Immich.

### Redis Settings

| Variable | Type | Default | Required |
|----------|------|---------|----------|
| `immich_docker_deploy_redis` | bool | `true` | ❌ |
| `immich_docker_redis_version` | raw | `9` | ❌ |
| `immich_docker_redis_image` | string | `docker.io/valkey/valkey:{{ immich_docker_redis_version }}` | ❌ |

**Details:**
- `immich_docker_deploy_redis`: Deploy a Redis instance for Immich. Set to `false` if using an external Redis instance.
- `immich_docker_redis_version`: Redis/Valkey Docker image version for Immich.
- `immich_docker_redis_image`: Redis/Valkey Docker image for Immich.

<!-- END ANSIBLE ROLE VARIABLES -->

## Example Playbooks

**Bare Minimum:**

```yml
---
- name: Deploy Immich
  hosts: server
  roles:
    - role: tigattack.immich_docker
      vars:
        immich_docker_db_password: your_secure_password_here
```

**With a custom data volume mount:**

```yml
---
- name: Deploy Immich
  hosts: server
  roles:
    - role: tigattack.immich_docker
      vars:
        immich_docker_db_password: your_secure_password_here
        immich_docker_data_use_docker_volume: true
        immich_docker_data_volume:
          name: immich_data
          driver: local
          driver_options:
            type: cifs
            device: //nas/photos
            o: addr=nas,username=immich,password=secret,dir_mode=0770,file_mode=0660
```

**With external PostgreSQL and Redis:**

```yml
---
- name: Deploy Immich
  hosts: server
  roles:
    - role: tigattack.immich_docker
      vars:
        immich_docker_db_password: your_secure_password_here
        immich_docker_deploy_postgres: false
        immich_docker_deploy_redis: false
        immich_docker_env_vars:
          DB_HOSTNAME: external-postgres.example.com
          DB_PORT: 5432
          REDIS_HOSTNAME: external-redis.example.com
          REDIS_PORT: 6379
```

**With NVIDIA ML and transcoding acceleration:**

```yml
---
- name: Deploy Immich with full NVIDIA acceleration
  hosts: server
  roles:
    - role: tigattack.immich_docker
      vars:
        immich_docker_db_password: your_secure_password_here
        immich_docker_machine_learning_acceleration: cuda
        immich_docker_transcoding_acceleration: nvenc
```

## License

MIT
