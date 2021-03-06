# DNS Lots-of-Lookups (DNSLOL)

[![Build Status](https://travis-ci.org/letsencrypt/dns-lots-of-lookups.svg?branch=master)](https://travis-ci.org/letsencrypt/dns-lots-of-lookups)
[![GolangCI](https://golangci.com/badges/github.com/letsencrypt/dns-lots-of-lookups.svg)](https://golangci.com/r/github.com/letsencrypt/dns-lots-of-lookups)


DNS Lots-of-Lookups (DNSLOL) is a small command line utility suitable for doing
large-scale DNS lookups quickly. In particular DNSLOL is tailored towards doing
comparative analysis of the results of doing these lookups through differently
configured recursive DNS resolvers.

## Install

DNSLOL is packaged as a [Go module](https://github.com/golang/go/wiki/Modules),
allowing you to clone the source outside of your `$GOPATH` and install using the
vendored dependencies.

1. [Set up Go 1.11+](https://golang.org/doc/install) and your `$GOPATH`
1. `git clone github.com/letsencrypt/dns-lots-of-lookups`
1. `cd dns-lots-of-lookups`
1. `go -mod=vendor install ./...`

## Usage

1. Increase the `ulimit` for the number of open files for your session to match
   your desired `-parallel` setting. E.g. for a `-parallel of 4000` run:

```bash
   ulimit nofile 4000
```

1. Start the lookup process. In this example we configure `dnslol` to perform `A`
   and `TXT` queries for every domain in `input_domains.txt`, using two local
   recursive resolvers (one on port `1053`, and one on `1054`). It will
   gradually spawn new goroutines every `15s`, up to a maximum of `4000`
   goroutines. Prometheus metrics will be exported on the debug address,
   `http://127.0.0.1:6363/metrics`.

```bash
   dnslol \
    -checkA \
    -checkTXT \
    -servers 127.0.0.1:1053,127.0.0.1:1054 \
    -parallel 4000 \
    -spawnInterval 15s \
    -debugAddr 127.0.0.1:6363 \
    < input_domains.txt
```

## Metrics

DNSLOL exports several [Prometheus](https://prometheus.io/) metrics on the configured `-debugAddr` that can be used to monitor the performance and results of a `dnslol` run.

| Metric Name      | Metric Type   | Labels    | Description           |
| ---------------- |---------------|-----------|:----------------------|
| `lookup_results` | Counter Vec   | `result`  | Result count per query rcode or error result |
| `attempts`       | Counter       |           | Number of lookup attempts made |
| `successes`      | Counter       |           | Number of lookup successes |
| `queryTime`      | SummaryVec    | `type`    | Query duration (seconds) per type |
| `commandLine`    | GaugeVec      | `line`    | Command line invocation of the `dnslol` tool |
