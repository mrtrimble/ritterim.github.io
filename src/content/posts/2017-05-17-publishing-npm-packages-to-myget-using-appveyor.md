---
layout: post
title: "Publishing npm Packages to MyGet Using AppVeyor"
date: 2017-05-16 17:00:00
tags:
- AppVeyor
- npm
- MyGet
categories:
- development
twitter_text: "Publishing npm Packages to MyGet Using AppVeyor"
authors: Ken Dale
---

Previously, I wrote about [Publishing to npm Using AppVeyor]({% post_url 2017-02-28-publishing-to-npm-using-appveyor %}). This is a similar post, but specifically geared toward publishing an npm package to [MyGet](https://myget.org/).

## Setup environment variable

In the AppVeyor web application, add a `MYGET_TOKEN` environment variable using a token from MyGet. **Be sure to encrypt this token in AppVeyor!**

## AppVeyor configuration in repository

Add or update the AppVeyor configuration using the following example:

## **appveyor.yml**

```yaml
environment:
  matrix:
    - nodejs_version: STABLE
      configuration: publish
    - nodejs_version: LTS

install:
  - ps: Install-Product node $env:nodejs_version

before_build:
  # Output useful info for debugging.
  - node --version
  - npm --version

build_script:
  - ./build.cmd

cache:
  - node_modules -> package.json
  - '%APPDATA%\npm-cache'

test: off

deploy_script:
  - ps: |
      if ($ENV:CONFIGURATION -eq "publish")
      {
        "registry=https://www.myget.org/F/«feed-name-here»/npm/`r`n//www.myget.org/F/«feed-name-here»/npm/:_authToken=`$`{MYGET_TOKEN`}" | Out-File (Join-Path $ENV:APPVEYOR_BUILD_FOLDER ".npmrc") -Encoding UTF8
        iex "npm pack"
        iex "npm publish"
      }
    on:
      branch: master
```

After implementing this in the `master` branch, check MyGet to see if the publish was successful *(assuming a new version was published)*.
