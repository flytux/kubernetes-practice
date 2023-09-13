# Nerdctl save images 

---

```bash
nerdctl -n k8s.io save -o all-local-images-in-namespace.tar $(nerdctl -n k8s.io image ls --format "{{.Repository}}:{{.Tag}}" | grep -v "none")
```
