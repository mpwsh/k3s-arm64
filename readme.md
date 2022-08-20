# TODO
- Add action to update manifest versions.

Example:
```bash
latest=$(curl -s https://api.github.com/repos/cert-manager/cert-manager/releases/latest | jq -r .tarball_url | cut -d/ -f8 | sed "s/v//g")
echo $latest
```
