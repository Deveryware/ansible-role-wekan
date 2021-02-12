# Ansible Role: Wekan

An Ansible role that builds and configures [Wekan](https://wekan.github.io/), an open source kanban.

Three directories are used during the lifecycle of a build: `wekan_src_directory`, `wekan_build_directory` and `wekan_run_directory`.
The first holds the Wekan source code fetched through `git(1)` with a shallow clone.
When in sync with latest upstream code its content is copied to `wekan_build_directory`.
NPM dependencies are then updated and Meteor is used to build the project and export the result to `wekan_run_directory`.

It means that both `wekan_src_directory` and `wekan_build_directory` can be located on a virtual memory file system.

## Requirements

The following software packages should be installed on the target server:

- nodejs
- npm
- git
- mongodb
- meteor

Due to multiple use of `become_user` it is also necessary to install the `acl` package (assuming Debian).

Additionnaly a reverse proxy such as nginx should be used to expose Wekan over https.

## Role Variables

| Variable                 | Description | Default |
|--------------------------|-------------|---------|
| `wekan_build_directory`  | Where to build wekan       | `/tmp/wekan-build` |
| `wekan_git_repository`   | Git repository URL         | `https://github.com/wekan/wekan.git` |
| `wekan_group`            | Name of wekan system group | `wekan` |
| `wekan_meteor_warehouse` | Path to Meteor warehouse   | mandatory |
| `wekan_proxy`            | Proxy URL                  | `` |
| `wekan_run_directory`    | Where to install wekan     | mandatory |
| `wekan_src_directory`    | Where to host wekan source | `/tmp/wekan-src` |
| `wekan_user`             | Name of wekan system user  | `wekan` |
| `wekan_version`          | Wekan release to install   | mandatory |
| `wekan_options`          | List of wekan options      | See bellow |
| `wekan_mongo_url`        | A mongodb:// URL           | mandatory |

Note: `wekan_proxy` is only used during build time for fetching NPM and Meteor dependencies.

### `wekan_options`

This is a list of environment variables used to configure Wekan.

```yaml
wekan_options:
  - "INTERNAL_LOG_LEVEL=info"
  - "LDAP_AUTHENTIFICATION=false"
  - "MONGO_URL='{{ wekan_mongo_url }}'"
  - "PORT=8080"
  - "ROOT_URL='{{ wekan_url }}'"
```

To enable LDAP authentification use these options:

```
LDAP_AUTHENTIFICATION_PASSWORD={{ ldap_password }}
LDAP_AUTHENTIFICATION=true
LDAP_AUTHENTIFICATION_USERDN={{ ldap_authdn }}
LDAP_BACKGROUND_SYNC=false
LDAP_BASEDN={{ ldap_basedn }}
LDAP_CA_CERT=
LDAP_CONNECT_TIMEOUT=1000
LDAP_EMAIL_FIELD=mail
LDAP_EMAIL_MATCH_ENABLE=false
LDAP_ENABLE=true
LDAP_ENCRYPTION=ssl
LDAP_FULLNAME_FIELD=cn
LDAP_GROUP_FILTER_ENABLE=false
LDAP_GROUP_FILTER_GROUP_ID_ATTRIBUTE=
LDAP_GROUP_FILTER_GROUP_MEMBER_ATTRIBUTE=
LDAP_GROUP_FILTER_GROUP_MEMBER_FORMAT=
LDAP_GROUP_FILTER_GROUP_NAME=
LDAP_GROUP_FILTER_OBJECTCLASS=
LDAP_HOST={{ ldap_host }}
LDAP_IDLE_TIMEOUT=10000
LDAP_INTERNAL_LOG_LEVEL=info
LDAP_LOG_ENABLED=true
LDAP_LOGIN_FALLBACK=false
LDAP_MERGE_EXISTING_USERS=true
LDAP_PORT=636
LDAP_RECONNECT=true
LDAP_REJECT_UNAUTHORIZED=false
LDAP_SEARCH_PAGE_SIZE=0
LDAP_SEARCH_SIZE_LIMIT=0
LDAP_SYNC_ADMIN_GROUPS=
LDAP_SYNC_ADMIN_STATUS=false
LDAP_SYNC_GROUP_ROLES=false
LDAP_SYNC_USER_DATA_FIELDMAP='{"cn":"fullname","uid":"username","mail":"mail"}'
LDAP_SYNC_USER_DATA=true
LDAP_TIMEOUT=10000
LDAP_UNIQUE_IDENTIFIER_FIELD=
LDAP_USER_ATTRIBUTES=
LDAP_USER_AUTHENTICATION=false
LDAP_USER_AUTHENTICATION_FIELD=uid
LDAP_USERDN={{ ldap_userdn }}
LDAP_USERNAME_FIELD=uid
LDAP_USER_SEARCH_FIELD=uid
LDAP_USER_SEARCH_FILTER=(&(objectclass=inetOrgPerson))
LDAP_USER_SEARCH_SCOPE=sub
LDAP_UTF8_NAMES_SLUGIFY=true
```

## Dependencies

An ansible role dedicated to the installation of the JavaScript framework [Meteor](https://www.meteor.com).

An ansible role dedicated to the configuration of MongoDB such as [UnderGreen/
ansible-role-mongodb](https://github.com/UnderGreen/ansible-role-mongodb)

## Example Playbook

```yaml
- hosts: wekan
  vars:
    wekan_meteor_warehouse: /srv/meteor/.meteor
    wekan_mongo_url: 'mongodb://{{ mongodb_user_admin_name }}:{{ mongodb_user_admin_password }}@localhost:27017/{{ mongodb_database_name }}'
    wekan_run_directory: /srv/wekan
    wekan_url: https://wekan.example.org
    wekan_version: "v4.22"
  roles:
     - role: undergreen.mongodb
     - role: deveryware.meteor
     - role: deveryware.wekan
```

## License

ISC

## Contributing

[Github pull requests](https://github.com/Deveryware/ansible-role-wekan) are gladly accepted.

## Author Information

Tristan Le Guern <tristan.leguern-presta@deveryware.com> for Deveryware.
