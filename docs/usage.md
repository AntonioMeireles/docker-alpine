# Usage

## Packages

Replacing your current base image with the Docker Alpine Linux image usually requires updating the package names to the corresponding ones in the [Alpine Linux package index][packageindex]. We use the `apk` command to manage packages. It works similar to `apt` or `yum`.

An example installing the `nginx` package would be `apk add --update nginx`. The `--update` flag fetches the current package index before adding the package. We don't ship the image with a package index (since that can go stale fairly quickly).

## Example

Here is a full example `Dockerfile` that installs the Python runtime and some build dependencies:

```
FROM gliderlabs/alpine:3.1

RUN apk add --update \
    python \
    python-dev \
    py-pip \
    build-base \
  && pip install virtualenv \
  && rm -rf /var/cache/apk/*

WORKDIR /app

ONBUILD COPY . /app
ONBUILD RUN virtualenv /env && /env/bin/pip install -r /app/requirements.txt

EXPOSE 8080
CMD ["/env/bin/python", "main.py"]
```

## Convenience Cleanup

This image contains a small unofficial wrapper script that assists in the cleanup of the package index after installing packages. A great minimalist cleans up after ones self. Thus, the `apk-install` script was born. Here is another simple `Dockerfile` that installs the `nginx` package and removes package index data:

```
FROM gliderlabs/alpine:3.1

RUN apk-install nginx

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Virtual Packages

Another great `apk add` feature for cleanup is the concept of virtual packages using the `--virtual` or `-t` switch. Packages added under this virtual name can then be removed as one group. An example use of this would be removing a group of build dependencies all at once:

```
FROM gliderlabs/alpine:3.1

WORKDIR /myapp
COPY . /myapp

RUN apk-install python py-pip openssl ca-certificates
RUN apk-install --virtual build-dependencies python-dev build-base wget \
  && pip install -r requirements.txt \
  && python setup.py install \
  && apk del build-dependencies

CMD ["myapp", "start"]
```

## Additional Information

Check out the [Alpine Linux package management documentation][apk] for more information and usage of `apk`.

[apk]: http://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management#Update_the_Package_list
[packageindex]: http://forum.alpinelinux.org/packages
