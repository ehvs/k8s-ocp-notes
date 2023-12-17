# DNS

By default, containers receive their DNS configuration file (/etc/resolv.conf) from their host (the node)

## Troubleshooting

- Check name resolution

```
export URL=https://myexample.com

curl -sS -I -k -w "status_code: %{http_code}, dns_resolution: %{time_namelookup}, tcp_established: %{time_connect}, total: %{time_total} \n" -o /dev/null $URL

curl -sS -I -k -w "status_code: %{http_code}, dns_resolution: %{time_namelookup}, tcp_established: %{time_connect}, total: %{time_total}, http_version: %{http_version}, time_redirect: %{time_redirect} \n" -o /dev/null $URL

Flags

-S, --show-error    Show error even when -s is used
-s, --silent        Silent mode
-I, --head          Show document info only
-k, --insecure      Allow insecure server connections when using SSL
-w, --write-out <format> Use output FORMAT after completion

https://everything.curl.dev/usingcurl/verbose/writeout
```
