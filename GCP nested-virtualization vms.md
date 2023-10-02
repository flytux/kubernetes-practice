# Create nested virtualization vms

```bash
gcloud compute instances create kw01 --enable-nested-virtualization --min-cpu-platform="Intel Haswell" --project=kw-poc-01  --zone=us-central1-a --machine-type=n1-standard-16 \
--create-disk=image=projects/debian-cloud/global/images/debian-11-bullseye-v20230912,mode=rw,size=100
```
