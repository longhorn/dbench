# dbench
Benchmark Kubernetes persistent disk volumes with `fio`: Read/write IOPS, bandwidth MB/s and latency.

This REPO is forked from [leeliu/dbench](https://github.com/leeliu/dbench), and is for Longhorn volume raw block device benchmarks.

# Usage

1. Deploy Longhorn [official document](https://longhorn.io/docs/1.0.1/deploy/) and **make sure there is no longhorn volume before this test**.
2. Download [dbench.yaml](https://raw.githubusercontent.com/longhorn/dbench/longhorn/dbench.yaml)
3. Deploy Dbench using: `kubectl apply -f dbench.yaml`
4. Once deployed, the Dbench Job will:
    * provision a Persistent Volume of `2Gi` (default) using `storageClassName: longhorn` (default)
    * run a series of `fio` tests on the newly provisioned disk
    * currently there are 9 tests, 15s per test - total runtime is ~2.5 minutes
4. Follow benchmarking progress using: `kubectl logs -f job/dbench` (empty output means the Job not yet created, or `storageClassName` is invalid, see Troubleshooting below)
5. At the end of all tests, you'll see a summary that looks similar to this:
```
==================
= Dbench Summary =
==================
Random Read/Write IOPS: 75.7k/59.7k. BW: 523MiB/s / 500MiB/s
Average Latency (usec) Read/Write: 183.07/76.91
Sequential Read/Write: 536MiB/s / 512MiB/s
Mixed Random Read/Write IOPS: 43.1k/14.4k
```
6. Once the tests are finished, clean up using: `kubectl delete -f dbench.yaml` and that should deprovision the persistent disk and delete it to minimize storage billing.

## Notes / Troubleshooting

* If the Persistent Volume Claim is stuck on Pending, it's likely you didn't specify a valid Storage Class. Double check using `kubectl get storageclasses`. Also check that the volume size of `2Gi` (default) is available for provisioning.
* It can take some time for a Persistent Volume to be Bound and the Kubernetes Dashboard UI will show the Dbench Job as red until the volume is finished provisioning.
* Uncomment the command in from [dbench.yaml](https://raw.githubusercontent.com/longhorn/dbench/longhorn/dbench.yaml) and then login to the pod and run `/docker-entrypoint.sh fio` directly to debug any issue.
* A list of all `fio` tests are in [docker-entrypoint.sh](https://raw.githubusercontent.com/longhorn/dbench/longhorn/docker-entrypoint.sh).

## License

* MIT
