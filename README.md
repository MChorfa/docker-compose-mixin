# Docker Compose Mixin for Porter

This is a mixin for Porter that provides the Docker Compose (docker-compose) CLI.

[![Build Status](https://dev.azure.com/deislabs/porter/_apis/build/status/porter-docker-compose?branchName=master)](https://dev.azure.com/deislabs/porter/_build/latest?definitionId=32&branchName=master)

## Mixin Declaration

To use this mixin in a bundle, declare it like so:

```yaml
mixins:
- docker-compose
```

## Mixin Syntax

See the [docker-compose CLI Command Reference](https://docs.docker.com/docker-compose/reference/) for the supported commands.

```yaml
docker-compose:
  description: "Description of the command"
  arguments:
  - arg1
  - arg2
  flags:
    FLAGNAME: FLAGVALUE
    REPEATED_FLAG:
    - FLAGVALUE1
    - FLAGVALUE2
  suppress-output: false
  outputs:
    - name: NAME
      jsonPath: JSONPATH
    - name: NAME
      path: SOURCE_FILEPATH
```

### Suppress Output

The `suppress-output` field controls whether output from the mixin should be
prevented from printing to the console. By default this value is false, using
Porter's default behavior of hiding known sensitive values. When 
`suppress-output: true` all output from the mixin (stderr and stdout) are hidden.

Step outputs (below) are still collected when output is suppressed. This allows
you to prevent sensitive data from being exposed while still collecting it from
a command and using it in your bundle.

### Outputs

The mixin supports `jsonpath` and `path` outputs.


#### JSON Path

The `jsonPath` output treats stdout like a json document and applies the expression, saving the result to the output.

```yaml
outputs:
- name: NAME
  jsonPath: JSONPATH
```

For example, if the `jsonPath` expression was `$[*].id` and the command sent the following to stdout: 

```json
[
  {
    "id": "1085517466897181794",
    "name": "my-vm"
  }
]
```

Then then output would have the following contents:

```json
["1085517466897181794"]
```

#### File Paths

The `path` output saves the content of the specified file path to an output.

```yaml
outputs:
- name: kubeconfig
  path: /root/.kube/config
```

---

## Examples

### Docker Compose Up

```yaml
docker-compose:
  description: "Docker Compose Up"
  arguments:
    - up
    - -d
  flags:
    timeout: 30
```

### Docker Compose Down

```yaml
docker-compose:
  description: "Docker Compose Down"
  arguments:
    - down
    - --remove-orphans
  flags:
    timeout: 30
```

See full bundle examples in the `examples` directory.

## Invocation

Use of this mixin requires opting-in to Docker host access via a Porter setting.  See the Porter [documentation](https://porter.sh/configuration/#allow-docker-host-access) for further details.

Here we opt-in via the CLI flag, `--allow-docker-host-access`:

```shell
$ porter install --allow-docker-host-access
Building bundle ===>
Copying porter runtime ===>
Copying mixins ===>
Copying mixin docker-compose ===>

Generating Dockerfile =======>

Writing Dockerfile =======>

Starting Invocation Image Build =======>
installing docker-compose...
executing install action from docker-compose (bundle instance: docker-compose)
Docker docker-compose up
app_web_1 is up-to-date
app_redis_1 is up-to-date
execution completed successfully!
```