---
layout: "registry"
page_title: "Terraform Registry - Publishing Providers"
sidebar_current: "docs-registry-provider-publishing"
description: |-
  Publishing Providers to the Terraform Registry
---

# Publishing Providers

Anyone can publish and share a provider by signing into the Registry using their GitHub account and following a few easy steps.

This page describes how to prepare a [Terraform Provider](/docs/plugins/provider.html) for publishing, and how to publish a prepared provider using the Registry's interface.

## Preparing your Provider

### Writing a Provider

Providers published to the Terraform Registry are written and built in the same way as other Terraform providers. A variety of resources are available to help our contributors build a quality integration:

- [Writing a custom provider – full tutorial](https://learn.hashicorp.com/tutorials/terraform/provider-setup)
- [How to build a provider – Video](https://www.youtube.com/watch?v=2BvpqmFpchI)
- [Sample provider developed by a HashiCorp partner](https://blog.container-solutions.com/write-terraform-provider-part-1)
- Example providers for reference:
    - [AWS](https://github.com/terraform-providers/terraform-provider-aws)
    - [AzureRM](https://github.com/terraform-providers/terraform-provider-azurerm)
- [Contributing to Terraform guidelines](/docs/extend/community/contributing.html)

~> **Important:** In order to be detected by the Terraform Registry, all provider repositories on GitHub must match the pattern `terraform-provider-{NAME}`, and the repository must be public.

### Documenting your Provider

Your provider should contain an overview document (index.md), as well as a doc for each resource and data-source. See [Documenting Providers](./docs.html) for details about how to ensure your provider documentation renders properly on the Terraform Registry.

-> **Note:** In order to test how documents will render in the Terraform Registry, you can use the [Terraform Registry Doc Preview Tool](https://registry.terraform.io/tools/doc-preview).

### Creating a GitHub Release

Publishing a provider requires at least one version be available on GitHub Releases. The tag must be a valid [Semantic Version](https://semver.org/) preceded with a `v` (for example, `v1.2.3`).

Terraform CLI and the Terraform Registry follow the Semantic Versioning specification when detecting a valid version, sorting versions, solving version constraints, and choosing the latest version. Prerelease versions are supported (available if explicitly defined but not chosen automatically) with a hyphen (-) delimiter, such as `v1.2.3-pre`.

We have a list of [recommend OS / architecture combinations](/docs/registry/providers/os-arch.html) for which we suggest most providers create binaries.

~> **Important:** Avoid modifying or replacing an already-released version of a provider, as this will cause checksum errors for users when attempting to download the plugin. Instead, if changes are necessary, please release as a new version.

#### Using GoReleaser locally

GoReleaser is a tool for building Go projects for multiple platforms, creating a checksums file, and signing the release. It can also upload your release to GitHub Releases.

1. Install [GoReleaser](https://goreleaser.com) using the [installation instructions](https://goreleaser.com/install/).
1. Copy the [.goreleaser.yml file](https://github.com/hashicorp/terraform-provider-scaffolding/blob/master/.goreleaser.yml) from the [hashicorp/terraform-provider-scaffolding](https://github.com/hashicorp/terraform-provider-scaffolding) repository.
1. Cache the password for your GPG private key with `gpg --armor --detach-sign` (see note below).
1. Set your `GITHUB_TOKEN` to a [Personal Access Token](https://github.com/settings/tokens) that has the **public_repo** scope.
1. Tag your version with `git tag v1.2.3`.
1. Build, sign, and upload your release with `goreleaser release --rm-dist`.

-> GoReleaser does not support signing binaries with a GPG key that requires a passphrase. Some systems may cache your GPG passphrase for a few minutes. If you are unable to cache the passphrase for GoReleaser, please use the manual release preparation process below, or remove the signature step from GoReleaser and sign it prior to moving the GitHub release from draft to published.

#### Manually Preparing a Release

If for some reason you're not able to use GoReleaser to build, sign, and upload your release, you can create the required assets by following these steps, or encode them into a Makefile or shell script.

The release must meet the following criteria:

* There are 1 or more zip files containing the built provider binary for a single architecture
    * The binary name is `terraform-provider-{NAME}_v{VERSION}`
    * The archive name is `terraform-provider-{NAME}_{VERSION}_{OS}_{ARCH}.zip`
* There is a `terraform-provider-{NAME}_{VERSION}_SHA256SUMS` file, which contains a sha256 sum for each zip file in the release.
    * `shasum -a 256 *.zip > terraform-provider-{NAME}_{VERSION}_SHA256SUMS`
* There is a `terraform-provider-{NAME}_{VERSION}_SHA256SUMS.sig` file, which is a valid GPG signature of the `terraform-provider-{NAME}_{VERSION}_SHA256SUMS` file using the keypair.
    * `gpg --detach-sign terraform-provider-{NAME}_{VERSION}_SHA256SUMS`
* Release is finalized (not a private draft).

## Publishing to the Registry

### Signing in

Before publishing a provider, you must first sign in to the Terraform Registry with a GitHub account (see [Signing into the Registry](/docs/registry/index.html#creating-an-account)). The GitHub account used must have the following permission scopes on the provider repository you’d like to publish. Permissions can be verified by going to your [GitHub Settings](https://github.com/settings/connections/applications/) and selecting the Terraform Registry Application under Authorized OAuth Apps.

![screenshot: terraform registry github oauth required permissions](./images/github-oauth-permissions.png)

### Preparing and Adding a Signing Key

All provider releases are required to be signed, thus you must provide HashiCorp with the public key for the GPG keypair that you will be signing releases with. The Terraform Registry will validate that the release is signed with this key when publishing each version, and Terraform will verify this during `terraform init`.

To export your public key in ASCII-armor format, use the following command:

```console
$ gpg --armor --export "{Key ID or email address}"
```

#### Individuals

If you would like to publish a provider under your GitHub username (not a GitHub organization), you may add your GPG key directly to the Terraform Registry by going to [User Settings > Signing Keys](https://registry.terraform.io/settings/gpg-keys).

#### Organizations

In order to publish a provider under a GitHub organization, your public key must be added to the Terraform Registry by a HashiCorp employee. Please send us an email to terraform-registry@hashicorp.com, including the information below, and one of us will get back to you shortly. For convenience, you may also use this <a href="mailto:terraform-registry@hashicorp.com?subject=Request%20to%20add%20GPG%20Key%20for%20%3CProvider%20Name%3E&body=Hello%2C%0D%0A%0D%0APlease%20add%20the%20following%20GPG%20key%20for%20my%20Terraform%20Provider%3A%0D%0A-%20GitHub%20organization%20(namespace)%3A%0D%0A-%20Link%20to%20provider%20repository%3A%20%0D%0A-%20GPG%20Key%3A%20(paste%20here%20or%20attach)">Email template</a>.

- GitHub organization (namespace):
- Link to provider repository: 
- GPG Key: (paste here or attach)

### Publishing Your Provider

In the top-right navigation, select [Publish > Provider](https://registry.terraform.io/publish/provider) to begin the publishing process. Follow the prompts to select the organization and repository you would like to publish.

#### Terms of Use

Anything published to the Terraform Registry is subject to our terms of use. A copy of the terms are available for viewing at https://registry.terraform.io/terms
