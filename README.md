# tether-autolaunchd-apt

Subscribe from Kali or Debian with:

```bash
REPO_URL="https://raw.githubusercontent.com/realagiorganization/tether-autolaunchd-apt/main"
echo "deb [trusted=yes] ${REPO_URL} ./" \
  | sudo tee /etc/apt/sources.list.d/tether-autolaunchd.list
sudo apt-get update
sudo apt-get install tether-autolaunchd
```
