---
title: "Linux commands"
weight: 1
description: >
  Helpful linux reference.
---

Write to a file from with cat:

```bash
$ cat << EOF > filename
# enter text
> EOF
```

Setting and unsetting environment variables:
```bash
$ export VARIABLE_NAME=new-variable-name
$ unset VARIABLE_NAME
```

Create dirs and files quickly:
```bash
$ mkdir -p /tmp/testdir/{text,logs}
$ touch /tmp/testdir/text/{text1,text2,text3}.txt
$ touch /tmp/testdir/logs/{log1,log2,log3}.log
```

Copy multiple files from a directory: 
```bash
$ cp ../tool/{add.go,go.mod} .
```

Creating `cron` job:
```bash
$ crontab -e # opens visual editor
$ 
```
Switch back to the previous working directory:
```bash
$ cd -
```


`time` executes an application and logs to the console how long it takes to run:
```bash
$ time ./colstats -op avg -col 3 testdata/example.csv testdata/example2.csv 
233.84

real	0m0.005s
user	0m0.006s
sys	0m0.000s
```
In the preceding example, `real` shows the total elapsed time.

The `tee` command logs command output to STDOUT and writes it to a file:
```
$ go test -bench . -benchtime=10x -run ^$ | tee benchresults00.txt
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 546183149 ns/op
PASS
ok  	colstats	6.067s

$ cat benchresults00.txt 
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 546183149 ns/op
PASS
ok  	colstats	6.067s
```

> TODO Learn how cURL works

```shell
curl -L -XPOST -d '{"task":"Task 1"}' -H 'Content-Type: application/json' http://localhost:8080/todo



```

## Compress files

Unzip `.zip` files with `unzip`. Do not include a directory if you want to unzip it to the current working directory:
```bash
$ unzip file.zip -p /unzip/to/this/dir
```

Extract contents of a tar file with `tar`:
```bash
$ tar -xzvf filename.tar.gz -C targetDir/
```

View compressed file info: 
```bash
$ gzip -l *
         compressed        uncompressed  ratio uncompressed_name
               1229                3047  61.1% experiment_toolid_test.go
               1021                2106  53.3% overlaydir_test.go
                696                1320  49.8% reboot_test.go
               2946                6473  55.0% (totals)
```

## cURL

Use the `-i` flag to display HTTP response headers with the body:

```shell
$ curl -i localhost:4000/v1/healthcheck
```

`-d` flag

The following command annotates the HTTP response with the total time for the command to complete:

```shell
$ curl -w '\nTime: %{time_total}s \n' localhost:4000/v1/movies/1
```

https://blog.josephscott.org/2011/10/14/timing-details-with-curl/

## xargs TODO

https://man7.org/linux/man-pages/man1/xargs.1.html

```shell
$ xargs -I % -P8 curl -X PATCH -d '{"runtime": "97 mins"}' "localhost:4000/v1/movies/4" < <(printf '%s\n' {1..8})
```