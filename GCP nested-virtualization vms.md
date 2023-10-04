# Create nested virtualization vms

```bash
gcloud compute instances create kw01-demo \
    --project=kw-poc-01 \
    --zone=us-central1-a \
    --machine-type=n1-standard-16 \
    --enable-nested-virtualization --min-cpu-platform="Intel Haswell" \
    --service-account=265842716061-compute@developer.gserviceaccount.com \
    --create-disk=auto-delete=yes,boot=yes,device-name=kw01-demo,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20230918,mode=rw,size=200,type=projects/kw-poc-01/zones/us-central1-a/diskTypes/pd-balanced
```
