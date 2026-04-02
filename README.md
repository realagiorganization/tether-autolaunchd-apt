# tether-autolaunchd-apt

Signed APT archive for stable and release-candidate `tether-autolaunchd` packages.

Stable channel:

```bash
KEYRING="/usr/share/keyrings/tether-autolaunchd-archive-keyring.gpg"
REPO_ROOT="https://raw.githubusercontent.com/realagiorganization/tether-autolaunchd-apt/main"
curl -fsSL "${REPO_ROOT}/tether-autolaunchd-archive-keyring.gpg" \
  | sudo tee "${KEYRING}" >/dev/null
echo "deb [signed-by=${KEYRING}] ${REPO_ROOT}/stable ./" \
  | sudo tee /etc/apt/sources.list.d/tether-autolaunchd.list
sudo apt-get update
sudo apt-get install tether-autolaunchd
```

Release candidate channel:

```bash
KEYRING="/usr/share/keyrings/tether-autolaunchd-archive-keyring.gpg"
REPO_ROOT="https://raw.githubusercontent.com/realagiorganization/tether-autolaunchd-apt/main"
curl -fsSL "${REPO_ROOT}/tether-autolaunchd-archive-keyring.gpg" \
  | sudo tee "${KEYRING}" >/dev/null
echo "deb [signed-by=${KEYRING}] ${REPO_ROOT}/rc ./" \
  | sudo tee /etc/apt/sources.list.d/tether-autolaunchd-rc.list
sudo apt-get update
sudo apt-get install tether-autolaunchd
```
