Creating a new package is simple: create a new directory and [author resources]:

```shell
$ mkdir awesomeapp
$ cd awesomeapp
# Create resources in awesome/
```

For convenience, you can use `pkg init` command to create a minimal `Kptfile` and `README` files:

```shell
$ cd awesomeapp
$ kpt pkg init
writing Kptfile
writing README.md
```

> Refer to the [command reference][init-doc] for more details.

The `info` section of the `Kptfile` contains some optional package metadata you may want to set.
These fields are not consumed by any functionality in kpt:

```yaml
apiVersion: kpt.dev/v1alpha2
kind: Kptfile
metadata:
  name: awesomeapp
info:
  description: Awesomeapp solves all the world's problems in half the time.
  site: awesomeapp.example.com
  emails:
    - jack@example.com
    - jill@example.com
  license: Apache-2.0
  keywords:
    - awesome-tech
    - world-saver
```

[author resources]: /book/03-packages/03-editing-a-package
[init-doc]: /reference/pkg/init/