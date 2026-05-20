# Elastic Search
examples

● Quick single-node setup for local dev (security disabled for simplicity):
```sh
  docker run -d --name elasticsearch \
    -p 9200:9200 -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    -e "ES_JAVA_OPTS=-Xms1g -Xmx1g" \
    docker.elastic.co/elasticsearch/elasticsearch:9.4.1

  # Then verify:

  curl http://localhost:9200
  ```

  Notes:
  - Pick a version tag that matches your client (replace 8.15.0 as needed).
  - For persistent data, add -v esdata:/usr/share/elasticsearch/data.
  - If the container exits immediately, it's usually the vm.max_map_count limit — fix
   with sudo sysctl -w vm.max_map_count=262144 on the host (in WSL2, run inside the
  WSL distro).
  - Drop xpack.security.enabled=false for production; you'll then need to set/retrieve the elastic user password.



```sh
# posting install
● Recommended path is pipx so it's isolated
   and on your PATH.

  # 1. Install pipx if you don't have it
  sudo apt update
  sudo apt install -y pipx
  pipx ensurepath

  # 2. Restart your shell (or: source
  ~/.bashrc)
  exec $SHELL

  # 3. Install Posting
  pipx install posting

  # 4. Run it
  posting

  If pipx install posting fails with a
  Python version error, Posting requires
  Python 3.11+. Ubuntu 22.04 ships 3.10.
  Fix:

  sudo add-apt-repository
  ppa:deadsnakes/ppa
  sudo apt update
  sudo apt install -y python3.12
  python3.12-venv
  pipx install --python python3.12 posting
```

```sh
posting --collection posting --env posting/envs/local.env
```

```sh

  Suggested scheme for ES

  cluster/
    health
    stats
    settings
  indices/
    list
    create
    delete
    mapping-get
  docs/
    index
    get
    update
    delete
  search/
    match-all
    by-field
    aggregations

  For the request you likely just made (GET /_cluster/health)

  cluster/health

  Rule of thumb

  - Folder = resource group (cluster, indices, docs, search).
  - Filename = action (health, create, delete, by-email).
  - Lowercase + hyphens — easier in a shell, plays nice with git.
  - Skip the HTTP verb in the name — Posting shows it in the sidebar already.

  If it's truly a one-off, just cluster/health is fine. You can rename/move later by
  editing the file on disk.
```