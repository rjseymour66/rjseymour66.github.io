---
title: "Learn with tests"
# linkTitle: ""
weight: 1
description: >
  [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests)
---

## Install go

This script will get the lastest version of Go and install it in `/usr/local`:

```bash
#!/bin/bash

# Upgrades the Go binary to the version specified as the first argument passed
# to this script.

# Check if the version argument was passed
if [ -z "$1" ]; then
	echo "Usage: $0 <go-version>"
	echo "Example: $0 1.24.4"
	exit 1
fi

VERSION="$1"
TARBALL="go${VERSION}.linux-amd64.tar.gz"
URL="https://go.dev/dl/${TARBALL}"

echo "Removing previous installation from /usr/local/go..."
rm -rf /usr/local/go

echo "Downloading go tarball from $URL..."
wget "$URL"

echo "Extracting the tarball to /usr/local..."
tar -C /usr/local -xzf "${TARBALL}"

echo "Cleaning up (deleting the tarball)..."
rm -v ${TARBALL}

echo
echo
echo "Verify the installation with 'go version'"
```

## go mod

https://go.dev/doc/modules/gomod-ref

If you plan to distribute your application, you need to tell others where your code is available for download. Thats what `go mod` does--it gives the name of the module and the download URL:

```bash
go mod init <path/to/module-name.com>
```

## Writing Tests

When you write tests, you are using the compiler as a feedback mechanism. Here is the feedback loop:
1. Write a test.
2. Write code to make the compiler pass.
3. Write another test.
4. Run the test, see that it fails and make sure the error message is meaningful.
5. Write code to make the compiler pass.
6. Refactor.

This makes sure that you are writing tested code with relative tests that are easier to debug when they fail.

### Placeholder strings

https://pkg.go.dev/fmt#hdr-Printing
