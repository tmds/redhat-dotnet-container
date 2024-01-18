# redhat-dotnet-container

This repo contains prototypes for improving the UX when using Red Hat .NET container images with the .NET SDK.

## RedHat.Container NuGet package

Adding this NuGet package makes the SDK use Red Hat .NET base images instead of Microsoft .NET base images. To opt-out of this behavior, you can set `UseRedHatContainerImage` to `false`.
The image locations can be overridden by setting `RedHatAspNet60RuntimeImage`, `RedHatAspNet70RuntimeImage`, ... .

Overview of properties:

| Property                   | Description | Default value |
|----------------------------|-------------|---------------|
| UseRedHatContainerImage       | Use Red Hat .NET images.                   | `true` |
| RedHatAspNet60RuntimeImage | Location of the .NET 6 ASP.NET Core runtime image. | `registry.access.redhat.com/ubi8/dotnet-60-runtime:latest` |
| RedHatAspNet70RuntimeImage | Location of the .NET 7 ASP.NET Core runtime image. | `registry.access.redhat.com/ubi8/dotnet-70-runtime:latest` |
| RedHatAspNet80RuntimeImage | Location of the .NET 8 ASP.NET Core runtime image. | `registry.access.redhat.com/ubi8/dotnet-80-runtime:latest` |

### Adding the package to your project

You can add the latest CI package to your project using:

```
dotnet add package RedHat.Container --prerelease -s https://www.myget.org/F/tmds/api/v3/index.json
```

## dotnet-publish-image Tekton task

The `dotnet-publish-image` Tekton task publishes a container image from a .NET project.

Task workspaces:

| Workspace | Description   |
|-----------|---------------|
| source    | Source code.  |
| dockerconfig | Additional registry credentials. |
| dotnetconfig | Shared .NET configuration.<br>If a `setup` file is present, its contents get executed before running the script.<br>If a `Build.props` file is present, it will be loaded by the .NET SDK through CustomBeforeDirectoryBuildProps. |

Overview of parameters:

| Parameter         | Description | Default value |
|-------------------|-------------|---------------|
| PROJECT           | Path of the .NET project file. | |
| IMAGE_NAME        | Image registry to push.<br>Name of the image repository to push. When it does not include a registry, it is pushed to the internal cluster registry. If no namespace is included, the current namespace is prepended to the name. | |
| SDK_VERSION       | Tag of .NET SDK imagestream. | `latest` |
| DOTNET_NAMESPACE  | Namespace of the .NET imagestreams. Set to `$(context.taskRun.namespace)` to use the pipeline namespace. | `openshift` |
| ENV_VARS          | Environment variables. | |
| VERBOSITY         | MSBuild verbosity level. | `minimal` |

Overview of results:

| Result         | Description | Example |
|-------------------|-------------|---------------|
| IMAGE           | Fully qualified image name of the image. | `image-registry.openshift-image-registry.svc:5000/my-namespace/my-app@sha256:5de0f82b679a0f3741044aa5db8a1cc92cc5449c08f7765ffe250f64842019fa` |
| IMAGE_DIGEST    | Digest of the image. | `sha256:5de0f82b679a0f3741044aa5db8a1cc92cc5449c08f7765ffe250f64842019fa` |

Environment variables:

| Variable         | Description | Value |
|------------------|-------------|-------|
| OpenShiftInternalRegistry | OpenShift internal registry hostname. | `image-registry.openshift-image-registry.svc:5000` |
| OpenShiftDotnetNamespace | Namespace that imports the .NET imagestreams. | `DOTNET_NAMESPACE` parameter |
| OpenShiftCurrentNamespace | Namespace where the pipeline runs. | `$(context.taskRun.namespace)` |
| RunningInTekton | Enables detecting if the project is built with Tekton. | `true` |

### Adding the task to your OpenShift namespace

```
oc apply -f https://raw.githubusercontent.com/tmds/redhat-dotnet-container/main/tekton/dotnet-publish-image.yaml
```

## dotnet-sdk Tekton task

The `dotnet-sdk` Tekton task allows to run a user-specified script that uses the .NET SDK.

Task workspaces:

| Workspace | Description   |
|-----------|---------------|
| source    | Source code.  |
| dockerconfig | Additional registry credentials. |
| dotnetconfig | Shared .NET configuration.<br>If a `setup` file is present, its contents get executed before running the script.<br>If a `Build.props` file is present, it will be loaded by the .NET SDK through CustomBeforeDirectoryBuildProps. |

Overview of parameters:

| Parameter         | Description | Default value |
|-------------------|-------------|---------------|
| SDK_VERSION       | Tag of .NET SDK imagestream. | `latest` |
| DOTNET_NAMESPACE  | Namespace of the .NET imagestreams. Set to `$(context.taskRun.namespace)` to use the pipeline namespace. | `openshift` |
| SCRIPT  | Bash script to run. | `dotnet --info` |
| ENV_VARS          | Environment variables. |

Environment variables:

| Variable         | Description | Value |
|------------------|-------------|-------|
| OpenShiftInternalRegistry | OpenShift internal registry hostname. | `image-registry.openshift-image-registry.svc:5000` |
| OpenShiftDotnetNamespace | Namespace that imports the .NET imagestreams. | `DOTNET_NAMESPACE` parameter |
| OpenShiftCurrentNamespace | Namespace where the pipeline runs. | `$(context.taskRun.namespace)` |
| RunningInTekton | Enables detecting if the project is built with Tekton. | `true` |

You can `dotnet publish` from the `SCRIPT` with the same semantics as `dotnet-publish-image` by using the `/p:PublishProfile=OpenShiftContainer` and setting `/p:OpenShiftImageName=<IMAGE_NAME>`.

### Adding the task to your OpenShift namespace

```
oc apply -f https://raw.githubusercontent.com/tmds/redhat-dotnet-container/main/tekton/dotnet-sdk.yaml
```
