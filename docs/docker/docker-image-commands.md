

# Docker Image Commands

## docker build

**Description：**Start a build

**Usage：**`docker buildx build [OPTIONS] PATH | URL | -`

**Aliases：**`docker buildx build`、`docker buildx b`

**Options：**

```shell
      --add-host strings              Add a custom host-to-IP mapping (format: "host:ip")
      --allow strings                 Allow extra privileged entitlement (e.g., "network.host", "security.insecure")
      --annotation stringArray        Add annotation to the image
      --attest stringArray            Attestation parameters (format: "type=sbom,generator=image")
      --build-arg stringArray         Set build-time variables
      --build-context stringArray     Additional build contexts (e.g., name=path)
      --builder string                Override the configured builder instance (default "default")
      --cache-from stringArray        External cache sources (e.g., "user/app:cache", "type=local,src=path/to/dir")
      --cache-to stringArray          Cache export destinations (e.g., "user/app:cache", "type=local,dest=path/to/dir")
      --cgroup-parent string          Set the parent cgroup for the "RUN" instructions during build
  -f, --file string                   Name of the Dockerfile (default: "PATH/Dockerfile")
      --iidfile string                Write the image ID to a file
      --label stringArray             Set metadata for an image
      --load                          Shorthand for "--output=type=docker"
      --metadata-file string          Write build result metadata to a file
      --network string                Set the networking mode for the "RUN" instructions during build (default "default")
      --no-cache                      Do not use cache when building the image
      --no-cache-filter stringArray   Do not cache specified stages
  -o, --output stringArray            Output destination (format: "type=local,dest=path")
      --platform stringArray          Set target platform for build
      --progress string               Set type of progress output ("auto", "plain", "tty"). Use plain to show container output (default "auto")
      --provenance string             Shorthand for "--attest=type=provenance"
      --pull                          Always attempt to pull all referenced images
      --push                          Shorthand for "--output=type=registry"
  -q, --quiet                         Suppress the build output and print image ID on success
      --sbom string                   Shorthand for "--attest=type=sbom"
      --secret stringArray            Secret to expose to the build (format: "id=mysecret[,src=/local/secret]")
      --shm-size bytes                Shared memory size for build containers
      --ssh stringArray               SSH agent socket or keys to expose to the build (format: "default|<id>[=<socket>|<key>[,<key>]]")
  -t, --tag stringArray               Name and optionally a tag (format: "name:tag")
      --target string                 Set the target build stage to build
      --ulimit ulimit                 Ulimit options (default [])

Experimental commands and flags are hidden. Set BUILDX_EXPERIMENTAL=1 to show them.
```

## docker history

**Description：**Show the history of an image

**Usage：**`docker history [OPTIONS] IMAGE`

**Aliases：**` docker image history`、`docker history`

**Options：**

```shell
  --format string   Format output using a custom template:
                    'table':            Print output in table format with column headers (default)
                    'table TEMPLATE':   Print output in table format using the given Go template
                    'json':             Print in JSON format
                    'TEMPLATE':         Print output using the given Go template.
                    Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -H, --human           Print sizes and dates in human readable format (default true)
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```

## docker inspect

**Description：**Return low-level information on Docker objects

**Usage：**`docker inspect [OPTIONS] NAME|ID [NAME|ID...]`

**Options：**

```shell
  -f, --format string   Format output using a custom template:
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
```

## docker load

**Description：**Load an image from a tar archive or STDIN

**Usage：**`docker load [OPTIONS]`

**Aliases：**`docker image load`、`docker load`

**Options：**

```shell
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output
```

## docker ls

**Description：**List images

**Usage：**`docker image ls [OPTIONS] [REPOSITORY[:TAG]]`

**Aliases：**`docker image ls`、`docker image list`、`docker images`

**Options：**

```shell
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```

## docker image prune

**Description：**Remove unused images

**Usage：**`docker image prune [OPTIONS]`

**Options：**

```shell
  -a, --all             Remove all unused images, not just dangling ones
      --filter filter   Provide filter values (e.g. "until=<timestamp>")
  -f, --force           Do not prompt for confirmation
```

## docker pull

**Description：**Download an image from a registry

**Usage：**`docker pull [OPTIONS] NAME[:TAG|@DIGEST]`

**Aliases：**`docker image pull`、`docker pull`

**Options：**

```shell
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
      --platform string         Set platform if server is multi-platform capable
  -q, --quiet                   Suppress verbose output
```

## docker push

**Description：**Upload an image to a registry

**Usage：**`docker push [OPTIONS] NAME[:TAG]`

**Aliases：**`docker image push` 、 `docker push`

**Options：**

```shell
  -a, --all-tags                Push all tags of an image to the repository
      --disable-content-trust   Skip image signing (default true)
  -q, --quiet                   Suppress verbose output	
```

## docker rmi

**Description：**Remove one or more images

**Usage：**`docker rmi [OPTIONS] IMAGE [IMAGE...]`

**Aliases：**`docker image rm`、`docker image remove`、`docker rmi`

**Options：**

```shell
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
```

## docker save

**Description：**Save one or more images to a tar archive (streamed to STDOUT by default)

**Usage：**` docker save [OPTIONS] IMAGE [IMAGE...]`

**Aliases：**`docker image save`、`docker save`

**Options：**

```shell
  -o, --output string   Write to a file, instead of STDOUT
```

## docker tag

**Description：**Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

**Usage：**`docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`

**Aliases：**`docker image tag`、`docker tag`



<script src="https://giscus.app/client.js"
        data-repo="wynhelloworld/blog-comments"
        data-repo-id="R_kgDOKruZpg"
        data-category="Announcements"
        data-category-id="DIC_kwDOKruZps4Ca2L0"
        data-mapping="url"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>

本站所有文章转发 **CSDN** 将按侵权追究法律责任，其它情况随意。