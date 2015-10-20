Goofys is a Filey-System interface to [S3](https://aws.amazon.com/s3/)

# Overview

Goofys allows you to mount an `S3` bucket as a filey system.

It's a Filey System instead of a File System because goofys strives
for performance first and POSIX second. Particularly things that are
difficult to support on `S3` or would translate into more than one
round-trip would either fail (random writes) or faked (no per-file
permission). Goofys does not have a on disk data cache, and
consistency model is close-to-open.

# Usage

Pre-built binaries are available [here](https://github.com/kahing/goofys/releases/).

```ShellSession
$ go get github.com/kahing/goofys
$ go install github.com/kahing/goofys
$ cat > ~/.aws/credentials
[default]
aws_access_key_id = AKID1234567890
aws_secret_access_key = MY-SECRET-KEY
$ $GOPATH/bin/goofys <bucket> <mountpoint>
```

Users can also configure credentials via the
[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
or the `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` environment variables.

# Benchmark

Using `--stat-cache-ttl 0 --type-cache-ttl 0` for goofys
`-ostat_cache_expire=1` for s3fs to simulate cold runs. Detail for the
benchmark can be found in
[bench.sh](https://github.com/kahing/goofys/blob/master/bench/bench.sh). [Raw data](https://github.com/kahing/goofys/blob/master/bench/)
is available as well. Test was run on an EC2 c4.xlarge in us-west-2a
connecting to a bucket in us-west-2.

operation |  s3fs  | goofys | speedup
----------| ------ | ------ | -------
create_files | 33.7+/-2.5 | 5.31+/-0.35 | 6.3+/-0.6x
rm_files | 6.6+/-0.6 | 3.1+/-0.4* | 2.12+/-0.32x
create_files_parallel | 29.4+/-1.7* | 2.45+/-0.30 | 12.0+/-1.6x
rm_files_parallel | 9.7+/-1.7 | 3.10+/-0.35 | 3.1+/-0.7x
ls_files | 9.6+/-1.4 | 0.173+/-0.029* | 55.2+/-12.3x
write_large_file | 38.4+/-6.2* | 11.7+/-1.8* | 3.3+/-0.7x
read_large_file | 22.0+/-6.7* | 17.1+/-1.1* | 1.3+/-0.4x
read_first_byte | 1.1+/-0.4 | 0.036+/-0.013* | 31.0+/-16.8x

(*) indicates the number of outliers removed

# License

Copyright (C) 2015 Ka-Hing Cheung

Licensed under the Apache License, Version 2.0

# Current Status

List of not yet implemented fuse operations:
  * in terms of syscalls
    * `readlink`
    * `chmod`/`utimes`/`ftruncate`
  * in terms of fuse functions
    * `ReadSymlink`
    * `SetInodeAttributes`

List of non-`POSIX` behaviors/limitations:
  * only sequential writes supported
  * does not support appending to a file yet
  * file mode is always `0644` for regular files and `0700` for directories
  * directories link count is always `2`
  * file owner is always the user running goofys
  * `ctime`, `atime` is always the same as `mtime`
  * cannot rename non-empty directories
  * `unlink` returns success even if file is not present
  * can only create files up to 50GB

# References

  * Data is stored on [Amazon S3](https://aws.amazon.com/s3/)
  * [Amazon SDK for Go](https://github.com/aws/aws-sdk-go)
  * Other related fuse filesystems
    * [s3fs](https://github.com/s3fs-fuse/s3fs-fuse): another popular filesystem for S3
    * [gcsfuse](https://github.com/googlecloudplatform/gcsfuse):
      filesystem for
      [Google Cloud Storage](https://cloud.google.com/storage/). Goofys
      borrowed some skeleton code from this project.
  * [S3Proxy](https://github.com/andrewgaul/s3proxy) is used for `go test`
  * [fuse binding](https://github.com/jacobsa/fuse), also used by `gcsfuse`
