# tether-autolaunchd-apt

Subscribe from Kali or Debian with:

```bash
KEYRING="/usr/share/keyrings/tether-autolaunchd-archive-keyring.gpg"
REPO_URL="https://raw.githubusercontent.com/realagiorganization/tether-autolaunchd-apt/main"
curl -fsSL "${REPO_URL}/tether-autolaunchd-archive-keyring.gpg" \
  | sudo tee "${KEYRING}" >/dev/null
echo "deb [signed-by=${KEYRING}] ${REPO_URL} ./" \
  | sudo tee /etc/apt/sources.list.d/tether-autolaunchd.list
sudo apt-get update
sudo apt-get install tether-autolaunchd
```

This repository is a signed flat APT archive. `Packages`, `Packages.gz`,
`Release`, `InRelease`, `Release.gpg`, and the exported archive keyring are
published together from the `tether-autolaunchd` release workflow.
