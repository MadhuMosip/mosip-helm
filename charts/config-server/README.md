# Config Server

Helm chart for installing for Config Server.

## Install

```console
$ helm repo add mosip https://mosip.github.io
$ helm install my-release mosip/config-server
```

## Add New Placeholder

* To add a new placeholder, we have to define it in the `_overides.tpl` file.
* In `overrides` section of `values.yaml`, add configmap or secret name in which new placeholder's value is defined.
  ```
  overrides:
    secrets:
       secretName: <secret-name>

    configmaps:
       configmapName: <configmap-name>
  ```
* Based upon new secrets or configmaps added, add respective section in  `_overides.tpl` file.   <br>
  `<key-name>`: is the actual keyname for new placeholder's value added in configmap/secrets. <br>
  `<PLACEHOLDER_NAME>`: is referred as `placeholder.name` by config-server in property files. <br>
  
  Example for new value in configmap:
  ```
  - name: SPRING_CLOUD_CONFIG_SERVER_OVERRIDES_<PLACEHOLDER_NAME>
    valueFrom:
      configMapKeyRef:
        name: {{ .Values.overrides.configmaps.configmapName }}
        key: <key-name>
  ```
  Example for new value in secret:
  ```
  - name: SPRING_CLOUD_CONFIG_SERVER_OVERRIDES_<PLACEHOLDER_NAME>
    valueFrom:
      secretKeyRef:
        name: {{ .Values.overrides.secrets.secretName }}
        key: <key-name>
  ```
  
### Enable config-server to pull configurations from local git repo.

Set the below configuration values as mentioned in the values.yaml file in-order to pull the configurations from local git repository
* Set `localRepo` enabled to `true`.
* Update the `spring.profiles.active` to `native` under localRepo.
* Update the `spring.cloud.config.server.native.search-locations` to `file:///var/lib/config_repo` as this is the mountDir where your local configurations are cloned/maintained.
* Update the `spring.cloud.config.server.accept-empty` to `true`.   # Server would return a HTTP 404 status, if the application is not found.By default, this flag is set to true.
* Update the `spring.cloud.config.server.git.force-pull` to `false`. # Spring Cloud Config Server makes a clone of the remote git repository and if somehow the local copy gets dirty (e.g. folder content changes by OS process) so Spring Cloud Config Server cannot update the local copy from remote repository but as our configurations are maintained locally we are setting this to `false`.
* Update the `spring.cloud.config.server.git.refreshRate` to `0`. # Setting up refresh rate to 5 seconds so that config server will check for updates in Git repo after every one minute, can be lowered down for production.
* Update the `spring.cloud.config.server.git.cloneOnStart` to `false`. # Adding provision to clone on start of server instead of first request but our configurations are stored in local so no need to clone the repository on start of server so setting it to `false`.

### Enable config-server to pull configurations from multiple sources.

In some scenarios, you may wish to pull configuration data from multiple environment repositories. To do so, you can enable the `composite profile` in your helm `values` YAML file, Composite Profiles in Spring Cloud Config Server provide a flexible mechanism for combining multiple profiles into a single effective profile. If, for example, you want to pull configuration data from a local repository as well as two Git repositories, you can set the following properties for your configuration server:

```
spring_profiles:
  enabled: true
  spring_profiles_active: composite
  spring_compositeRepos:
    - type: git     # Type "git" is to pull the configurations from remote git repository.
      uri: "https://github.com/mosip/inji-config"
      version: develop
      spring_cloud_config_server_git_cloneOnStart: true
      spring_cloud_config_server_git_force_pull: true
      spring_cloud_config_server_git_refreshRate: 5
    - type: git
      uri: "https://github.com/mosip/mosip-config"
      version: develop
      spring_cloud_config_server_git_cloneOnStart: true
      spring_cloud_config_server_git_force_pull: true
      spring_cloud_config_server_git_refreshRate: 5
    - type: native   # Type "native" is to pull the configurations from local git repository.
      uri: "file:///var/lib/config_repo"    # Dir path of local git repo
      version: develop
      spring_cloud_config_server_git_cloneOnStart: false   # This is set to "false" when type is "native".
      spring_cloud_config_server_git_force_pull: false     # This is set to "false" when type is "native".
      spring_cloud_config_server_git_refreshRate: 0    # This is set to "0" when type is "native".  
  spring_fail_on_composite_error: false
```

Using the above configuration, precedence is determined by the order in which repositories are listed under the composite key. In the above example, the git repository is listed first, so a value found in the git repository will override values found for the same property in the second configuration Git repository and third configuration local repository.

Note: Based on the user requiremnt the number of multiple sources from where configuration needs to be pulled can be updated as mentioned in the above code block.