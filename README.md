# Ansible Role: Keycloak Client Management

[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-blue.svg)](https://spdx.org/licenses/MIT-0.html)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-eclipse--slm.keycloak_client-blue.svg)](https://galaxy.ansible.com/eclipse-slm/keycloak_client)

An Ansible role for managing Keycloak resources including realms, clients, groups, and roles. This role provides a comprehensive set of tasks for automating Keycloak configuration through its Admin REST API.

## Table of Contents
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Tasks](#tasks)
  - [Authentication](#authentication)
  - [Realms](#realms)
  - [Clients](#clients)
  - [Groups](#groups)
  - [Roles](#roles)
- [Dependencies](#dependencies)
- [Example Playbook](#example-playbook)
- [Testing](#testing)
- [License](#license)
- [Author Information](#author-information)

## Features

This role supports managing the following Keycloak resources:

- **Authentication**: Obtain admin access tokens
- **Realms**: Create and list Keycloak realms
- **Clients**: Create clients with protocol mappers, retrieve client secrets, and store configuration in Consul
- **Groups**: Create groups, nested groups, and assign users to groups
- **Roles**: Create realm roles and assign them to users or service accounts

## Requirements

- Ansible 2.2 or higher
- Python 3.x
- Network access to Keycloak instance
- Keycloak admin credentials
- For Consul integration: Consul instance with valid token

## Role Variables

All variables are optional and should be defined based on your specific use case. The role uses a prefix-based naming convention:

- `keycloak.*` - Keycloak instance configuration
- `ENV.KEYCLOAK.*` - Keycloak environment variables (admin credentials, realm name)
- `ENV.CONSUL.*` - Consul environment variables (for config storage)

### Key Configuration Variables

```yaml
# Keycloak instance URL
keycloak:
  url:
    external: "https://keycloak.example.com"

# Admin credentials
ENV:
  KEYCLOAK:
    ADMIN_USERNAME: "admin"
    ADMIN_PASSWORD: "admin-password"
    REALM: "master"

# Consul configuration (optional)
ENV:
  CONSUL:
    SCHEME:
      INTERNAL: "http"
    HOST:
      INTERNAL: "consul.example.com"
    PORT:
      INTERNAL: 8500
```

## Tasks

### Authentication

#### `authentication/get_admin_access_token.yml`
Obtains an admin access token from Keycloak using the password grant type.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `ENV.KEYCLOAK.ADMIN_USERNAME` - Admin username
- `ENV.KEYCLOAK.ADMIN_PASSWORD` - Admin password

**Outputs:**
- `keycloak_access_token` - The obtained access token

### Realms

#### `realms/create_realm.yml`
Creates a new Keycloak realm.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm` - Realm representation (see [Keycloak API docs](https://www.keycloak.org/docs-api/22.0.1/rest-api/index.html#RealmRepresentation))

#### `realms/get_realms.yml`
Lists all Keycloak realms.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token

**Outputs:**
- `keycloak_realms` - List of realms

### Clients

#### `clients/create_client.yml`
Creates a new Keycloak client with optional protocol mappers.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_keycloak_client` - Client definition with:
  - `client_id` - Client identifier (required)
  - `public_client` - Boolean for public/confidential client (required)
  - `service_accounts_enabled` - Boolean for service accounts (required)

**Optional Variables:**
- `_keycloak_client_store_config_consul` - Store config in Consul (default: true)
- `_keycloak_client.protocol_mappers.groups` - Enable Groups mapper
- `_keycloak_client.protocol_mappers.audience` - Enable Audience mapper
- `_keycloak_client.assign_client_role_to_service_account` - Assign client role to service account

**Outputs:**
- `client_uuid` - UUID of the created client
- `keycloak_client` - Client details

#### `clients/get_client_by_client_id.yml`
Retrieves a client by its client ID.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_keycloak_client_id` - Client ID to search for

**Outputs:**
- `keycloak_client` - Client details

#### `clients/get_client_secret.yml`
Retrieves the client secret for a confidential client.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_client_id` - Client ID

**Outputs:**
- `client_secret` - Client secret value

#### `clients/get_oidc-keycloak-json.yml`
Retrieves the OIDC configuration in Keycloak JSON format.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `client_id` - Client ID

**Outputs:**
- `keycloak_oidc_json` - OIDC configuration JSON

#### `clients/store_client_config_consul.yml`
Stores client configuration in Consul KV store.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `client_id` - Client ID
- `ENV.CONSUL.SCHEME.INTERNAL` - Consul scheme
- `ENV.CONSUL.HOST.INTERNAL` - Consul host
- `ENV.CONSUL.PORT.INTERNAL` - Consul port
- `_consul_token` - Consul token

### Groups

#### `groups/create_group.yml`
Creates a new group in a realm.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_keycloak_group` - Group representation with `name` field

#### `groups/get_groups.yml`
Lists all groups in a realm.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name

**Outputs:**
- `_keycloak_groups` - List of groups

#### `groups/create_child_group.yml`
Creates a child group under a parent group.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_keycloak_parent_group_id` - Parent group UUID
- `_keycloak_group` - Child group representation

#### `groups/create_child_group_and_assign_user.yml`
Creates a child group and assigns a user to it.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_username` - Username to assign
- `_keycloak_parent_group_name` - Parent group name
- `_keycloak_child_group` - Child group representation

### Roles

#### `roles/create_realm_role.yml`
Creates a new realm role.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_keycloak_realm_role` - Role definition with `name` field

#### `roles/get_realm_roles.yml`
Lists all realm roles.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name

**Outputs:**
- `keycloak_realm_roles` - List of realm roles

#### `roles/assign_realm_role_to_user.yml`
Assigns a single realm role to a user.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_username` - Username
- `_realm_role_name` - Role name to assign

#### `roles/assign_realm_roles_to_user.yml`
Assigns multiple realm roles to a user.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_keycloak_user` - User definition with:
  - `username` - Username (required)
  - `realmRoles` - List of role names to assign (required)

#### `roles/assign_client_role_to_service_account.yml`
Assigns a client role to a service account.

**Required Variables:**
- `keycloak.url.external` - Keycloak instance URL
- `_keycloak_access_token` - Admin access token
- `_keycloak_realm_name` - Target realm name
- `_keycloak_client` - Client with service accounts enabled
- `_client_id` - Client ID with role to assign
- `_client_role_name` - Client role name to assign

## Dependencies

This role has no external dependencies beyond Ansible core modules. It uses:
- `ansible.builtin.uri` - For API calls
- `ansible.builtin.set_fact` - For variable management
- `community.general.consul_kv` - For Consul integration (optional)

## Example Playbook

### Basic Realm and Client Creation

```yaml
---
- name: Configure Keycloak
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    keycloak:
      url:
        external: "http://localhost:8080"
    ENV:
      KEYCLOAK:
        ADMIN_USERNAME: "admin"
        ADMIN_PASSWORD: "admin"
        REALM: "my-realm"
    
    realm:
      name: "my-realm"
      display_name: "My Realm"
      enabled: true
    
    realm_role:
      name: "my-role"
    
    realm_group:
      name: "my-group"
    
    realm_client:
      client_id: "my-client"
      public_client: false
      service_accounts_enabled: true
      protocol_mappers:
        groups: true
        audience: true

  tasks:
    - name: Get Keycloak admin access token
      ansible.builtin.include_role:
        name: eclipse-slm.keycloak_client
        tasks_from: authentication/get_admin_access_token.yml

    - name: Create Keycloak realm
      ansible.builtin.include_role:
        name: eclipse-slm.keycloak_client
        tasks_from: realms/create_realm.yml
      vars:
        _keycloak_realm: "{{ realm }}"

    - name: Create realm role
      ansible.builtin.include_role:
        name: eclipse-slm.keycloak_client
        tasks_from: roles/create_realm_role.yml
      vars:
        _keycloak_realm_name: "{{ realm.name }}"
        _keycloak_realm_role:
          name: "{{ realm_role.name }}"

    - name: Create group
      ansible.builtin.include_role:
        name: eclipse-slm.keycloak_client
        tasks_from: groups/create_group.yml
      vars:
        _keycloak_realm_name: "{{ realm.name }}"
        _keycloak_group:
          name: "{{ realm_group.name }}"

    - name: Create client
      ansible.builtin.include_role:
        name: eclipse-slm.keycloak_client
        tasks_from: clients/create_client.yml
      vars:
        _keycloak_realm_name: "{{ realm.name }}"
        _keycloak_client_store_config_consul: false
        _keycloak_client:
          client_id: "{{ realm_client.client_id }}"
          public_client: "{{ realm_client.public_client }}"
          service_accounts_enabled: "{{ realm_client.service_accounts_enabled }}"
          protocol_mappers:
            groups: "{{ realm_client.protocol_mappers.groups }}"
            audience: "{{ realm_client.protocol_mappers.audience }}"
```

### Assign Roles to User

```yaml
- name: Assign roles to user
  hosts: localhost
  connection: local
  vars:
    keycloak:
      url:
        external: "http://localhost:8080"
    ENV:
      KEYCLOAK:
        ADMIN_USERNAME: "admin"
        ADMIN_PASSWORD: "admin"
        REALM: "my-realm"
  
  tasks:
    - name: Get admin access token
      ansible.builtin.include_role:
        name: eclipse-slm.keycloak_client
        tasks_from: authentication/get_admin_access_token.yml

    - name: Assign realm roles to user
      ansible.builtin.include_role:
        name: eclipse-slm.keycloak_client
        tasks_from: roles/assign_realm_roles_to_user.yml
      vars:
        _keycloak_realm_name: "{{ ENV.KEYCLOAK.REALM }}"
        _keycloak_user:
          username: "john.doe"
          realmRoles:
            - "role1"
            - "role2"
```

## Testing

This role uses [Molecule](https://molecule.readthedocs.io/) for testing with Docker containers:

```bash
# Run all tests
molecule test

# Converge only
molecule converge

# Verify only
molecule verify

# Destroy instance
molecule destroy
```

The test suite covers:
- Admin token authentication
- Realm creation and retrieval
- Realm role creation and assignment
- Group creation and user assignment
- Client creation with protocol mappers

## License

MIT-0 - See [LICENSE](https://spdx.org/licenses/MIT-0.html) for details.

## Author Information

- **Author**: Benjamin Goetz
- **Company**: Fraunhofer IPA
- **Namespace**: eclipse-slm
- **Role Name**: keycloak_client

This role is part of the [SLM (Software Lifecycle Management)](https://github.com/eclipse-slm) project.
