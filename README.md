## Generating disk images of (desktop) operating systems

While most Linux distributions provide pre-built cloud disk images (e.g., [Debian](https://cloud.debian.org/images/cloud/), [Rocky Linux](https://rockylinux.org/download)) for server installations, pre-built disk images for desktop installation are lacking[^windows-vm]. Various open-source tools exist to automatically generate (and customize, e.g., with pre-installed packages) such disk images: 

[^windows-vm]: For Windows, Microsoft is providing a pre-built [development image](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/).

- [virt-install](https://github.com/virt-manager/virt-manager/blob/main/virt-install) ([example in this repository](virt-install)) (potentially combined with [Ansible](https://github.com/ansible/); see [virssh](virt-install/virssh) on how to get the IP address of a VM running with virt-install)
- [ubuntu-image](https://github.com/canonical/ubuntu-image) ([example in this repository](ubuntu-image))
- Packer [1.9.5](https://releases.hashicorp.com/packer/1.9.5/)[^packer]
- [mkosi](https://github.com/systemd/mkosi)
- [virt-builder](https://github.com/libguestfs/guestfs-tools/tree/master/builder)
- [Image Builder](https://github.com/osbuild)
- [debos](https://github.com/go-debos/debos)
- [oz-install](https://github.com/clalancette/oz/blob/master/oz-install)
- [openstack-debian-image](https://salsa.debian.org/openstack-team/debian/openstack-debian-images)
- [arch-boxes](https://gitlab.archlinux.org/archlinux/arch-boxes) (and [build_in_archiso_vm.sh](https://gitlab.archlinux.org/archlinux/ci-scripts/-/blob/master/scripts/build_in_archiso_vm.sh))

[^packer]: Later Packer versions [are licensed](https://www.hashicorp.com/license-faq#products-covered-by-bsl) as proprietary ["source-available software"](https://en.wikipedia.org/wiki/Source-available_software), which is worse than providing no source code at all because the source code's availability is used to threaten open-source projects with [claims of copyright infringement](https://opentofu.org/blog/our-response-to-hashicorps-cease-and-desist/).
Using Packer's non-cost version to ["compete"](https://www.hashicorp.com/license-faq#usage-limitations) with paid versions is forbidden by the proprietary license. Using Packer in GitLab CI/CD or GitHub Actions to generate publicly available disk images could potentially be characterized as competing with Packer's paid cloud version.

### How to download the images generated in this repository

Use [oras](https://oras.land/docs/installation):
```
curl -L "https://github.com/$(curl -L https://github.com/oras-project/oras/releases/ | grep -Eom1 '/[^"]*/oras_[^"]+_linux_amd64.tar.gz')" | tar xz -C ~/.local/bin oras

oras pull ghcr.io/opensourcevdi/disk-images:main-virt-install
oras pull ghcr.io/opensourcevdi/disk-images:main-ubuntu-image 
```

or the [get-oci-blob-url](get-oci-blob-url) from this repository to get a (time-limited) download URL:

```
./get-oci-blob-url ghcr.io/opensourcevdi/disk-images:main-virt-install
./get-oci-blob-url ghcr.io/opensourcevdi/disk-images:main-ubuntu-image
```

which you can, e.g., use with a web browser or combine with `curl`:

```
curl -Lo image.qcow2 "$(./get-oci-blob-url ghcr.io/opensourcevdi/disk-images:main-virt-install)"
curl -Lo image.qcow2 "$(./get-oci-blob-url ghcr.io/opensourcevdi/disk-images:main-ubuntu-image)"
```

## Saving large files from GitLab CI/CD/GitHub Actions

An ideal storage for providing large public artifacts, e.g., disk images, produced by GitLab CI/CD or GitHub Actions would 
- allow to host files with an unlimited file size
- for an indefinite time,
- provide the raw file/blob (as octet-stream),
- provide a digest (i.e., a cryptographic hash, e.g., SHA-256) of the file's content,
- be directly based on HTTP(S) (such that files can be linked and downloaded, e.g., via a web browser or `curl`),
- not require authentication for downloading (for uploading, it should be integrated into GitLab CI/CD or GitHub Actions),
- and allow for easy mirroring of the provided files to other locations.

The following public or self-hosted services fulfill these requirements to various degrees.

&nbsp; | Service | Maximum artifact size | Retention time | Container format | Provides digest of content | Access | Unauthenti&shy;cated access | Mirroring easily possible
--- | --- | --- | --- | --- | --- | --- | --- | ---
&nbsp; | GitLab CI/CD job artifacts | 1000 MiB [(on gitlab.com)](https://gitlab.com/help/instance_configuration#size-limits) | 90 days or latest for branch | .zip (custom URLs to download individual files available) | No | HTTP redirect | Yes | No
&nbsp; | GitHub Actions job artifacts | [unlimited](https://github.com/actions/upload-artifact/issues/9) | [90 days](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration#artifact-and-log-retention-policy) | .zip | No | HTTP redirect or GitHub API | No, requires GitHub account both via web site and via [GitHub API](https://docs.github.com/en/rest/actions/artifacts?apiVersion=2022-11-28#download-an-artifact); [workarounds](https://nightly.link/) exist but can be [brittle](https://nightly.link/#authorization) | No
⭐ | [GitLab generic package registry](https://docs.gitlab.com/ee/user/packages/generic_packages/#publish-a-package-file) | unlimited (on gitlab.com or [with correct settings](https://docs.gitlab.com/ee/user/packages/generic_packages/#internal-server-error-on-large-file-uploads-to-s3)) | indefinite | octet-stream | Yes, [SHA-256](https://docs.gitlab.com/ee/api/packages.html#list-package-files) | HTTP redirect | Yes | Somewhat ([custom API](https://docs.gitlab.com/ee/api/packages.html#list-packages))
&nbsp; | GitHub releases (associated with a Git tag) | 2 GiB | indefinite | octet-stream | No | HTTP redirect | Yes | No ([custom API](https://docs.github.com/en/rest/releases/releases?apiVersion=2022-11-28#list-releases) with rate limits)
&nbsp; | [GitLab pages](https://docs.gitlab.com/ee/user/project/pages/) | 1000 MiB [(on gitlab.com)](https://gitlab.com/help/instance_configuration#size-limits) | indefinite | octet-stream | No | HTTP(S) | Yes | No
&nbsp; | [GitHub pages](https://docs.github.com/en/pages) | [1 GB](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#usage-limits) | indefinite | octet-stream | No | HTTP(S) | Yes | No
&nbsp; | GitLab Git repository | 100 MiB [(on gitlab.com)](https://docs.gitlab.com/ee/user/free_push_limit.html) | indefinite | octet-stream | Yes, SHA-1 (SHA-256 [planned](https://git-scm.com/docs/hash-function-transition)) | HTTP(S) | Yes | Yes
&nbsp; | GitHub Git repository | [100 MiB](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github#file-size-limits) | indefinite | octet-stream | Yes, SHA-1 (SHA-256 [planned](https://git-scm.com/docs/hash-function-transition)) | HTTP(S) | Yes | Yes
(⭐) | GitLab Git LFS | 10 GB [(on gitlab.com)](https://docs.gitlab.com/ee/user/gitlab_com/index.html#account-and-limit-settings) | indefinite | octet-stream | Yes, [SHA-256](https://github.com/git-lfs/git-lfs/blob/main/docs/spec.md#user-content-the-pointer) | HTTP(S) or [Git LFS API](https://github.com/git-lfs/git-lfs/blob/main/docs/api/basic-transfers.md) | Yes | Yes
&nbsp; | GitHub Git LFS | [2 GB](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage#about-git-large-file-storage) | indefinite | octet-stream | Yes, [SHA-256](https://github.com/git-lfs/git-lfs/blob/main/docs/spec.md#user-content-the-pointer) | HTTP(S) or [Git LFS API](https://github.com/git-lfs/git-lfs/blob/main/docs/api/basic-transfers.md) | Yes | Yes
⭐ | GitLab container registry | unlimited (on [registry.&ZeroWidthSpace;gitlab.com](https://registry.gitlab.com)) | indefinite (unless [cleanup policy](https://docs.gitlab.com/ee/user/packages/container_registry/reduce_container_registry_storage.html#cleanup-policy) active) | octet-stream (or .tar.gz when not using [ORAS](https://oras.land/)) | Yes, [SHA-256](https://github.com/opencontainers/image-spec/blob/main/descriptor.md#digests) | [OCI distribution spec](https://github.com/opencontainers/distribution-spec)[^get-oci-blob-url] | Yes | [Yes](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#listing-tags)
⭐ | GitHub container registry ([ghcr.io](https://ghcr.io)) | unlimited | indefinite | octet-stream (or .tar.gz when not using [ORAS](https://oras.land/)) | Yes, [SHA-256](https://github.com/opencontainers/image-spec/blob/main/descriptor.md#digests) | [OCI distribution spec](https://github.com/opencontainers/distribution-spec)[^get-oci-blob-url] | Yes | [Yes](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#listing-tags)
(⭐) | Quay container registry ([quay.io](https://quay.io)) | unlimited | indefinite | octet-stream (or .tar.gz when not using [ORAS](https://oras.land/)) | Yes, [SHA-256](https://github.com/opencontainers/image-spec/blob/main/descriptor.md#digests) | [OCI distribution spec](https://github.com/opencontainers/distribution-spec)[^get-oci-blob-url] | Yes | [Yes](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#listing-tags)
&nbsp; | Docker Hub container registry ([registry-1.docker.io](https://registry.hub.docker.com)) | unlimited | indefinite | octet-stream (or .tar.gz when not using [ORAS](https://oras.land/)) | Yes, [SHA-256](https://github.com/opencontainers/image-spec/blob/main/descriptor.md#digests) | [OCI distribution spec](https://github.com/opencontainers/distribution-spec)[^get-oci-blob-url] | Yes (with [rate limits](https://docs.docker.com/docker-hub/download-rate-limit/)) | [Yes](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#listing-tags)
(⭐) | External S3 (object/blob) storage | unlimited | indefinite | octet-stream | Yes but usually [MD5-based](https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity.html#checking-object-integrity-md5) (SHA-256 [depending on provider](https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity.html#using-additional-checksums)) | HTTP(S) or [S3 API](https://docs.aws.amazon.com/AmazonS3/latest/API/API_Operations_Amazon_Simple_Storage_Service.html) | Yes | [Yes](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html)

[^get-oci-blob-url]: The [get-oci-blob-url](get-oci-blob-url) script provided in this repository can generate an (often time-limited) HTTP(S) URL to download a file/blob from an (OCI) container registry.

For all listed providers, unlimited storage is provided for free for public projects. However, GitLab is [planning](https://about.gitlab.com/pricing/#when-are-the-future-gitlab-com-namespace-storage-and-transfer-limits-applicable) to introduce a (total) storage limit of [5 GiB](https://docs.gitlab.com/ee/user/usage_quotas.html#namespace-storage-limit) in future.

