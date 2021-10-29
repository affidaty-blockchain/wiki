Development Server
==================

## General Information

**Service Account**

- ID: QmfZy5bvk7a3DQAjCbGNtmrPXWkyVvPrdnZMyBZ5q5ieKG
- contract: "service"

**Node Account**

- ID: QmamzDVuZqkUDwHikjHCkgJXhtgkbiVDTvTYb2aq6qfLbY
- contract "basic-asset"

**Available endpoints**

- nightly: rolling release from development branch.
- staging: stable version from master branch (with debug symbols).
- release: stable version from master branch.

---

* Core version: `https://t2.dev.trinci.net/<ENDPOINT>`
* REST endpoint: `https://t2.dev.trinci.net/<ENDPOINT>/api/v1`

To inspect published contracts:

```
curl https://t2.dev.trinci.net/<ENDPOINT>/api/v1/account/QmfZy5bvk7a3DQAjCbGNtmrPXWkyVvPrdnZMyBZ5q5ieKG:contracts | msgpack2json -d -b

```

Requires [msgpack2json](https://github.com/ludocode/msgpack-tools)
