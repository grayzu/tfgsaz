# Getting Started with Terraform v.12 on Azure

This repo for  guide at https://learn.hashicorp.com/terraform?track=azure#azure. The files here are the source for files in https://github.com/hashicorp/learn/tree/master/pages/terraform/azure.

This project is set up to build with [DocFX](https://dotnet.github.io/docfx/) for preview. When creating a PR to hashicorp, exclude the following files that are associated with docfx:
```
    /.gitignore
    /docfx.json
    /index.md
    /toc.yml
    /articles/toc.yml
```
To build a preview with DocFX:
1. Install docfx, same installation method as terraform, just one binary.
2. In the root /getting-started/ run `docfx docfx.json`
3. DocFX will create end up with a new `_site` folder. The index document is _site/index.html. 
NOTE: DocFX refreshes only the files that were modified, and it never removes orphan/deleted files. To see changes between iterations, you'll usually have to clear the browser cache. 
4. YOu can use `docfx serve` to host the site on localhost:8080
    
As of this writing I don't have a final destination for the contents of the /code folder -- TBD.

## TOC 

/articles/toc.yml

```
- name: Introduction
  href: intro.md
- name: Installing Terraform
  href: install.md
- name: Create Configuration
  href: configure.md
- name: Build Infrastructure
  href: build.md
- name: Change Infrastructure
  href: change.md
- name: Destroy Infrastructure
  href: destroy.md
- name: Resource Dependencies
  href: dependencies.md
- name: Provision
  href: provision.md
- name: Input Variables
  href: variables.md
- name: Output Variables
  href: outputs.md
- name: Modules
  href: modules.md
- name: Remote State Storage
  href: remote.md
- name: Next Steps
  href: next-steps.md
  ```
