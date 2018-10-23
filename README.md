# Helm push artifactory plugin

A Helm plugin to push helm charts to artifactory:
 
 * A version for artifactory of helm-push: https://github.com/chartmuseum/helm-push
 * Using a couple of things from Jfrog-cli-go: https://github.com/jfrog/jfrog-cli-go
 * And a bit of makefile magic from: https://github.com/helm/helm

## Install
Based on the version in `plugin.yaml`, release binary will be downloaded from GitHub:

```
$ helm plugin install https://github.com/belitre/helm-push-artifactory-plugin
Downloading and installing helm-push-artifactory v0.1.0 ...
https://github.com/belitre/helm-push-artifactory-plugin/releases/download/v0.1.0/helm-push-artifactory_v0.1.0_darwin_amd64.tar.gz
Installed plugin: push-artifactory
```

## Usage
__This plugin doesn't use repositories added through Helm CLI, in Artifactory those are virtual repositories. To push a chart we need to use a local repository URL__

Example
```
$ helm push-artifactory /my/chart/folder https://my-artifactory/my-local-repo --username username --password password
```

For all available plugin options, please run
```
$ helm push-artifactory --help
```

### Pushing a directory
Point to a directory containing a valid `Chart.yaml` and the chart will be packaged and uploaded:
```
$ cat mychart/Chart.yaml
name: mychart
version: 0.3.2
```
```
$ helm push-artifactory mychart/ https://my-artifactory/my-local-repo
Pushing mychart-0.3.2.tgz to https://my-artifactory/my-local-repo/mychart/mychart-0.3.2.tgz...
Done.
Reindex helm repository my-local-repo...
Reindex of helm repo my-local-repo was scheduled to run.
```

### Pushing with a custom version
The `--version` flag can be provided, which will push the package with a custom version.

Here is an example using the last git commit id as the version:
```
$ helm push-artifactory mychart/ --version="$(git log -1 --pretty=format:%h)" https://my-artifactory/my-local-repo
Pushing mychart-5abbbf28.tgz to https://my-artifactory/my-local-repo/mychart/mychart-5abbbf28.tgz...
Done.
Reindex helm repository my-local-repo...
Reindex of helm repo my-local-repo was scheduled to run.
```

### Push .tgz package
This workflow does not require the use of `helm package`, but pushing .tgz is still supported:
```
$ helm push mychart-0.3.2.tgz https://my-artifactory/my-local-repo
Pushing mychart-0.3.2.tgz to https://my-artifactory/my-local-repo/mychart/mychart-0.3.2.tgz...
Done.
Reindex helm repository my-local-repo...
Reindex of helm repo my-local-repo was scheduled to run.
```

### Push with path
You can set a path to push your chart in your Artifactory local repository:
```
$ helm push-artifactory mychart/ https://my-artifactory/my-local-repo --path organization
Pushing mychart-0.3.2.tgz to https://my-artifactory/my-local-repo/organization/mychart/mychart-0.3.2.tgz...
Done.
Reindex helm repository my-local-repo...
Reindex of helm repo my-local-repo was scheduled to run.
```

### Skip repository reindex
You can skip triggering the repository reindex:
```
$ helm push-artifactory mychart/ https://my-artifactory/my-local-repo --skip-reindex
Pushing mychart-0.3.2.tgz to https://my-artifactory/my-local-repo/mychart/mychart-0.3.2.tgz...
Done.
```

## Authentication
### Basic Auth
__The plugin will not use the auth info located in `~/.helm/repository/repositories.yaml` in order to authenticate.__

You can provide username and password through commmand line with `--username username --password password` or use the following environment variables for basic auth on push operations:
```
$ export HELM_REPO_USERNAME="myuser"
$ export HELM_REPO_PASSWORD="mypass"
```

### Access Token
You can provide an access token through command line with `--access-token my-token` or use the following env var:
```
$ export HELM_REPO_ACCESS_TOKEN="<token>"
```

If only the access token is supplied without any username, the plugin will send the token in the header:
```
Authorization: Bearer <token>
```

If a username is supplied with an access token, the plugin will use basic authentication, using the access token as password for the user.

### Api Key
You can provide an api key through command line with `--api-key my-key` or use the following env var:
```
$ export HELM_REPO_API_KEY="<api-key>"
```

If only the api key is supplied without any username, the plugin will send the api key in the header:
```
X-JFrog-Art-Api: <api-key>
```

If a username is supplied with an api key, the plugin will use basic authentication, using the api key as password for the user.

### TLS Client Cert Auth

If you need to setup your TLS cert authentication, the following options are available:

```
--ca-file string    Verify certificates of HTTPS-enabled servers using this CA bundle [$HELM_REPO_CA_FILE]
--cert-file string  Identify HTTPS client using this SSL certificate file [$HELM_REPO_CERT_FILE]
--key-file string   Identify HTTPS client using this SSL key file [$HELM_REPO_KEY_FILE]
--insecure          Connect to server with an insecure way by skipping certificate verification [$HELM_REPO_INSECURE]
```
