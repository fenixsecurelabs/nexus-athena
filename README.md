### Underground Nexus - Athena0
> Kali Linux Bleeding Edge Respository, v0.8.5

### Information

This is single component of the entire Underground Nexus. Kali Linux Bleeding Repository is built from scratch with using `kalilinux/kali-bleeding-edge:amd64` as the base image. This newly built Docker image has Wireshark, NMap, Metasploit Framework, and radare2 for your use.

To get started with Underground Nexus, you will need to already have built the Underground Nexus. There is a script already in place to pull the necessary Docker image and run within the dockerized environment.

I use a script on my PC to build out the Docker images. If you would like to build something similar to mine, you can use this script below.
```bash
VERSION=$(git log -1 --pretty=%h)
REPO="registry.gitlab.com/<USERNAME>/nexus-athena0:"
TAG="$REPO$VERSION"
LATEST="${REPO}latest"
BUILD_TIMESTAMP=$( date '+%F_%H:%M:%S' )

docker buildx build -t "$TAG" -t "$LATEST" --build-arg VERSION="$VERSION" --build-arg BUILD_TIMESTAMP="$BUILD_TIMESTAMP" . --no-cache --pull --push

# docker push "$TAG" 
# docker push "$LATEST"
```

So to the best of my ability, I have multiple Github Actions running to deploy separate Docker images to two different container registries:

1. GitLab Container Registry
2. GitHub Container Registry

There is also a GitHub Action to run `trivy` to scan the built Docker image and upload the `SARIF` results to GitHub Security Scanning Alerts.

### Cosign usage

To get started with using Cosign, see below.

Generate GitLab key-pair with Cosign. You will need to create a GITLAB_TOKEN from GitLab.

```bash
➜  nexus-kali-linux git:(main) ✗ cosign generate-key-pair gitlab://cyberphxv/nexus-athena0
Enter password for private key: 
Enter password for private key again: 
Password written to "COSIGN_PASSWORD" variable
Private key written to "COSIGN_PRIVATE_KEY" variable
Public key written to "COSIGN_PUBLIC_KEY" variable
Public key also written to cosign.pub
```

Sign both of the images with Cosign
```bash
cosign sign -a "repo=https://gitlab.com/cyberphxv/nexus-athena0" \
  -a commit=$(git rev-parse HEAD) \
  --key gitlab://cyberphxv/nexus-athena0 "$TAG"

Pushing signature to: registry.gitlab.com/cyberphxv/nexus-athena0

cosign sign -a "repo=https://gitlab.com/cyberphxv/nexus-athena0" \
  -a commit=$(git rev-parse HEAD) \
  --key gitlab://cyberphxv/nexus-athena0 "$LATEST" 
Pushing signature to: registry.gitlab.com/cyberphxv/nexus-athena0
```

Verify signed image with `cosign`.

```bash
cosign verify --key cosign.pub "$TAG" | jq .

Verification for registry.gitlab.com/cyberphxv/nexus-athena0:a271cb3 --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key
[
  {
    "critical": {
      "identity": {
        "docker-reference": "registry.gitlab.com/cyberphxv/nexus-athena0"
      },
      "image": {
        "docker-manifest-digest": "sha256:cde890708a5de8d9ac5be8b6555115282c4f2cda621a08b82778f99aec50045f"
      },
      "type": "cosign container image signature"
    },
    "optional": null
  },
  {
    "critical": {
      "identity": {
        "docker-reference": "registry.gitlab.com/cyberphxv/nexus-athena0"
      },
      "image": {
        "docker-manifest-digest": "sha256:cde890708a5de8d9ac5be8b6555115282c4f2cda621a08b82778f99aec50045f"
      },
      "type": "cosign container image signature"
    },
    "optional": {
      "repo": "https://gitlab.com/cyberphxv/nexus-athena0"
    }
  },
  {
    "critical": {
      "identity": {
        "docker-reference": "registry.gitlab.com/cyberphxv/nexus-athena0"
      },
      "image": {
        "docker-manifest-digest": "sha256:cde890708a5de8d9ac5be8b6555115282c4f2cda621a08b82778f99aec50045f"
      },
      "type": "cosign container image signature"
    },
    "optional": {
      "commit": "a271cb34acc62bef12e1bb459668c336276a66f5",
      "repo": "https://gitlab.com/cyberphxv/nexus-athena0"
    }
  }
]
```

Use `crane` with `cosign`.

```bash
crane manifest $(cosign triangulate "$TAG") | jq .
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 451,
    "digest": "sha256:27bdb64383f56167aa6f55a607e26fa5ee97b2d1c1089ae03353efd02d7e93d2"
  },
  "layers": [
    {
      "mediaType": "application/vnd.dev.cosign.simplesigning.v1+json",
      "size": 259,
      "digest": "sha256:17be2efb2c43a8bc7a99135c1fde0531f40f7f8baba0d3b22e997de2aacb3b9a",
      "annotations": {
        "dev.cosignproject.cosign/signature": "MEQCIAJKsRDQ8KnBG16mijLImqWu3OAMIu5xh3eFv7anE5wyAiA/CVcrttyUnTBmH53iiFNib8pwinf3jlWhs/8mjMROLA=="
      }
    },
    {
      "mediaType": "application/vnd.dev.cosign.simplesigning.v1+json",
      "size": 308,
      "digest": "sha256:fc008526a1fc9697ac232256d3f88830416a5c355dae0fe4bb137917267e61e6",
      "annotations": {
        "dev.cosignproject.cosign/signature": "MEUCIAgPzxLmQOZXSK/opCTrEaR54m2UKF2ixX5RRPUvh18VAiEA35Ckn9UT8EPSif6Mg4SmbVSWwSiWWp6flGAQku/ZRAU="
      }
    },
    {
      "mediaType": "application/vnd.dev.cosign.simplesigning.v1+json",
      "size": 360,
      "digest": "sha256:6ab78f4908edf5c7c3bf5a33a52fcae3be25f180a1c325ff0b204eb349d647b1",
      "annotations": {
        "dev.cosignproject.cosign/signature": "MEYCIQDE7p0ZYCtMlsChA6OKG2an158FyLW54atrq4aXs1ESGwIhAPTjaP2MXso6zNQ0ljOMSlltSZQUU7F1ZHGEH9LLt1Tr"
      }
    }
  ]
}
```

Use `syft` packge the newly built Docker image and then export it to `.spdx` file. Attach `sbom` to the docker image, and sign with your choice of `cosign` key. Lastly verify the newly signed `sbom` with your `cosign` public key.

```bash
syft packages "$TAG" -o spdx > athena0-latest.spdx

cosign attach sbom --sbom athena0-latest.spdx "$TAG"
WARNING: Attaching SBOMs this way does not sign them. If you want to sign them, use 'cosign attest -predicate athena0-latest.spdx -key <key path>' or 'cosign sign -key <key path> <sbom image>'.
Uploading SBOM file for [registry.gitlab.com/cyberphxv/nexus-athena0:a271cb3] to [registry.gitlab.com/cyberphxv/nexus-athena0:sha256-cde890708a5de8d9ac5be8b6555115282c4f2cda621a08b82778f99aec50045f.sbom] with mediaType [text/spdx].

cosign sign --key gitlab://cyberphxv/nexus-athena0 registry.gitlab.com/cyberphxv/nexus-athena0:sha256-cde890708a5de8d9ac5be8b6555115282c4f2cda621a08b82778f99aec50045f.sbom
Pushing signature to: registry.gitlab.com/cyberphxv/nexus-athena0

cosign verify --key cosign.pub registry.gitlab.com/cyberphxv/nexus-athena0:sha256-cde890708a5de8d9ac5be8b6555115282c4f2cda621a08b82778f99aec50045f.sbom

Verification for registry.gitlab.com/cyberphxv/nexus-athena0:sha256-cde890708a5de8d9ac5be8b6555115282c4f2cda621a08b82778f99aec50045f.sbom --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key

[{"critical":{"identity":{"docker-reference":"registry.gitlab.com/cyberphxv/nexus-athena0"},"image":{"docker-manifest-digest":"sha256:40922352623a7bca04472dcc6b388fa3424209515f6b47e1eeb86345a172e507"},"type":"cosign container image signature"},"optional":null}]
```