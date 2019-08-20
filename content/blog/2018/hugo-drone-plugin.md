---
title: A hugo plugin for your Drone CI pipeline
date: 2018-03-03T14:53:29.000Z
draft: false
tags:
  - Development
  - Programming
  - Hugo
  - Drone CI
  - Bash
categories:
  - Development
  - Continuous Integration
description: >-
  I have developed a plugin for the Continuous Delivery platform Drone.io. This
  makes publishing your statically generated web pages a breeze!
---

That I'm very attached to Drone is no secret! In an earlier article I introduced you to the static web pages generator Hugo, as well as my workflow with which I update this blog regularly.

Updating the files after a commit was possible in different ways. For example, you can generate and commit the static files, and then upload the created directory via SSH or (S)FTP. This is not very clean, because the generated files are not part of the project source code.

A better approach is to generate the static files based on the source files in the repository during the CI process. In my workflow I created an image based on Ubuntu, which contains the current Hugo binary files and executes them to generate the static files.

**`drone.yml`**

```
pipeline:
  publish:
    image: cynthek/hugo-deployment:latest
    secrets: [ SCP_USER, SCP_PASS, SCP_URL, SCP_PATH ]
    commands:
    - hugo
    - sshpass -p "$SCP_PASS" scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -r ./public/* $SCP_USER@$SCP_URL:$SCP_PATH
    when:
branch: [ master ]
```

The whole thing worked wonderfully, but had a few small disadvantages:

- The image was very big for the fact that only a small command should be executed.
- I didn't really use Drones YAML file correctly, because I only used the command parameter here instead of encapsulating the instructions via single parameters (Like Plugins should do).

That was reason enough for me to optimize the whole thing. I needed a plugin. Best of all, one that is much smaller than my Ubuntu-based image (~180MB!), one that is easy to integrate into the existing pipeline, is flexible and expandable and at the same time offers the greatest possible freedom in configuration.

So now I would like to introduce you to cbrgm/drone-hugo, a Hugo plugin for Drone CI!

# cbrgm/drone-hugo

[![GitHub release](https://img.shields.io/github/release/cbrgm/drone-hugo.svg)](https://github.com/cbrgm/drone-hugo/releases) [![Docker Pulls](https://img.shields.io/docker/pulls/cbrgm/drone-hugo.svg)](https://hub.docker.com/r/cbrgm/drone-hugo/tags/)

**Automatically create static web page files using Hugo within your drone pipeline!**

cbrgm/drone-hugo is:

- **Easy** to implement in your existing pipeline using `.drone.yml`
- **Small** 21mb image size
- **Highly configurable**

## Basic Usage with Drone CI

The example below demonstrates how you can use the plugin to automatically create static web page files using Hugo. **It's as easy as pie!**

```
pipeline:
  hugo:
    image: cbrgm/drone-hugo:latest
    validate: true
```

`validate` allows you to check your configuration file for errors before generating the files.

### Customize source, output, theme, config, layout OR content directory paths

You can customize the paths for e. g. the theme, layout, content directory and output directory and much more!

```
pipeline:
  hugo:
    image: cbrgm/drone-hugo:latest
+   config: path/to/config
+   content: path/to/content/
+   layout: path/to/layout
+   output: path/to/public
+   source: path/to/source
+   theme: path/themes/THEMENAME/
    validate: true
```

### Set hostname (and path) to the root

You can also define a base URL directly in the pipeline, which is used when generating the files.

```
pipeline:
  hugo:
    image: cbrgm/drone-hugo:latest
    config: path/to/config
    content: path/to/content/
    output: path/to/public
    source: path/to/source
    theme: path/themes/THEMENAME/
+   url: https://cbrgm.de
    validate: true
```

### Build sites and include expired, drafts or future content

You can set the `buildDrafts`, `buildExpired`, `buildFuture` settings to configure the generated files.

- `buildDrafts` - include content marked as draft
- `buildExpired` - include expired content
- `buildFuture` - include content with publishdate in the future

```
pipeline:
  hugo:
    image: cbrgm/drone-hugo:latest
+   buildDrafts: true
+   buildExpired: true
+   buildFuture: true
    config: path/to/config
    content: path/to/content/
    output: path/to/public
    source: path/to/source
    theme: path/themes/THEMENAME/
    url: https://cbrgm.de
    validate: true
```

### **Example**: Generate Hugo static files and publish them to remote directory using scp

Here is a short example of how to define a pipeline that automatically generates the static web page files with Hugo and then copies them to a remote server via scp. This makes publishing websites a breeze!

```
pipeline:
  build:
    image: cbrgm/drone-hugo:latest
    output: site # Output path
    validate: true
    when:
      branch: [ master ]
  publish:
    image: appleboy/drone-scp
    host: cbrgm.de
    username: webuser
    password: xxxxxxx
    port: 54321
    target: /var/www/ # Path to your web directory
    source: site/* # Copy all files from output path
```

You can also use secrets to hide credentials:

```
pipeline:
  build:
    image: cbrgm/drone-hugo:latest
    output: site # Output path
    validate: true
    when:
      branch: [ master ]
  publish:
    image: appleboy/drone-scp
+   secrets: [ ssh_username, ssh_password ]
    host: cbrgm.de
-   username: webuser
-   password: xxxxxxx
    port: 54321
    target: /var/www/ # Path to your web directory
    source: site/* # Copy all files from output path
```

## Basic Usage using a Docker Container

```bash
docker run --rm \
  -e PLUGIN_BUILDDRAFTS=false \
  -e PLUGIN_BUILDEXPIRED=false \
  -e PLUGIN_BUILDFUTURE=false \
  -e PLUGIN_CONFIG=false \
  -e PLUGIN_CONTENT=false \
  -e PLUGIN_LAYOUT=false \
  -e PLUGIN_OUTPUT=false \
  -e PLUGIN_SOURCE=false \
  -e PLUGIN_THEME=false \
  -e PLUGIN_OUTPUT=false \
  -e PLUGIN_VALIDATE=false \
  -v $(pwd):$(pwd) \
  -w $(pwd) \
  cbrgm/drone-hugo:latest
```

## Parameter Reference

`buildDrafts` - include content marked as draft<br>
`buildExpired` - include expired content<br>
`buildFuture` - include content with publishdate in the future<br>
`config` - config file (default is path/config.yaml|json|toml)<br>
`content` - filesystem path to content directory<br>
`layout` - filesystem path to layout directory `output` - filesystem path to write files to<br>
`source` - filesystem path to read files relative from<br>
`theme` - theme to use (located in /themes/THEMENAME/)<br>
`url` - hostname (and path) to the root<br>
`validate` - validate config file before generation

## Conclusion

I am very satisfied with the result and will use the plugin to generate my project documentation in the future. If you have any suggestions or wishes, please feel free to contact me at Github, or fork the project and create a pull request! I look forward to your feedback!

Cheers Chris
