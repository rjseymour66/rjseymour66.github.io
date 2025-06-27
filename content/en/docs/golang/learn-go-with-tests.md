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