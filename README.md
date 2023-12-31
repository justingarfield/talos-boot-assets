# talos-boot-assets

## Overview

This repository contains a [GitHub Actions](https://docs.github.com/en/actions) workflow that runs on a cronjob every 
day to check and see if a new official [Talos Linux](https://github.com/siderolabs/talos) release has been pushed.

If it detects a newer version is available _(compared to the tag(s) in this repo)_ it will use the 
[Talos Imager](https://github.com/siderolabs/talos/tree/v1.5.0/pkg/imager) to build new 
[Boot Assets](https://www.talos.dev/v1.5/talos-guides/install/boot-assets/) used in my 
[home-ops](https://github.com/justingarfield/home-ops) environment.

## Release Artifacts

Currently I output three artifacts from this workflow:

| Artifact Name | Description |
|-|-|
| `metal-rpi_generic-arm64.raw.xz` | Shows up under [Releases](https://github.com/justingarfield/talos-boot-assets/releases), used for Raspberry Pi 4 Raw Disk Image |
| `installer-amd64`                | Shows up under [Packages](https://github.com/justingarfield?tab=packages&repo_name=talos-boot-assets), used for Hyper-V VM installs / upgrades |
| `installer-arm64`                | Shows up under [Packages](https://github.com/justingarfield?tab=packages&repo_name=talos-boot-assets), used for Raspberry Pi 4 upgrades |

## Workflow

```mermaid
flowchart TD

subgraph check-for-new-release
    GetTalosRelease[Get Latest\nTalos\nRelease Version]
    GetBootAssetRelease[Get Latest\nBoot-Assets\nRelease Version]

    GetTalosRelease-->NewVersionAvail
    GetBootAssetRelease-->NewVersionAvail

    NewVersionAvail{Newer Talos\nversion available?}
end

NewVersionAvail -->|Yes| build-boot-assets
NewVersionAvail -->|No| End

subgraph build-boot-assets
    BuildInstallerAmd64[Build Talos\namd64 Installer\nContainer Image]
    BuildInstallerAmd64 --> BuildInstallerArm64[Build Talos\narm64 Installer\nContainer Image]
    BuildInstallerArm64 --> BuildRpi[Build Talos\nmetal-rpi_generic\nRaw Image]
    BuildRpi --> UploadArtifacts[Upload\nArtifacts]
end

UploadArtifacts --> create-release
UploadArtifacts --> push-container-image

subgraph create-release
    DownloadCreateRelease[Download\nArtifacts]
    DownloadCreateRelease --> CreateNewRelease[Create new\nGitHub Tag\nand Release]
end

subgraph push-container-image
    DownloadPushContainer[Download\nArtifacts]
    DownloadPushContainer --> SetupDockerBuildX[Setup Docker\nBuildx]
    SetupDockerBuildX --> LoginToGhcr[Login to\nghcr.io Registry]
    LoginToGhcr --> CranePush[Crane Push\nContainer Image]
end

CreateNewRelease --> End
CranePush --> End

End[End Workflow]
```
