# redhat-dotnet-container

This repo contains prototypes for improving the UX when using Red Hat .NET container images with the .NET SDK.

## RedHat.Container NuGet package

Adding this NuGet package makes the SDK use Red Hat .NET base images instead of Microsoft .NET base images. To opt-out of this behavior, you can set `UseRedHatContainerImage` to `false`.
The image locations can be overridden by setting `RedHatAspNet60RuntimeImage`, `RedHatAspNet70RuntimeImage`, ... .

Overview of properties:

| Property                   | Description | Default value |
|----------------------------|-------------|---------------|
| UseRedHatContainerImage       | Use Red Hat .NET images.                   | true |
| RedHatAspNet60RuntimeImage | Location of the .NET 6 ASP.NET Core runtime image. | registry.access.redhat.com/ubi8/dotnet-60-runtime:latest |
| RedHatAspNet70RuntimeImage | Location of the .NET 7 ASP.NET Core runtime image. | registry.access.redhat.com/ubi8/dotnet-70-runtime:latest |
| RedHatAspNet80RuntimeImage | Location of the .NET 8 ASP.NET Core runtime image. | registry.access.redhat.com/ubi8/dotnet-80-runtime:latest |

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

Overview of parameters:

| Parameter         | Description | Default value |
|-------------------|-------------|---------------|
| PROJECT           | Path of the .NET project file. | |
| IMAGE_NAME        | Image registry to push.<br>Name of the image repository to push. When it does not include a registry, it is pushed to the internal cluster registry. If no namespace is included, the current namespace is prepended to the name. | |
| SDK_VERSION       | Tag of .NET SDK imagestream. | latest |
| DOTNET_NAMESPACE  | Namespace of the .NET imagestreams. Set to '$(context.taskRun.namespace)' to use the pipeline namespace. | openshift |

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

Overview of parameters:

| Parameter         | Description | Default value |
|-------------------|-------------|---------------|
| SDK_VERSION       | Tag of .NET SDK imagestream. | latest |
| DOTNET_NAMESPACE  | Namespace of the .NET imagestreams. Set to '$(context.taskRun.namespace)' to use the pipeline namespace. | openshift |
| SCRIPT  | Bash script to run. | dotnet --info |

You can `dotnet publish` from the `SCRIPT` with the same semantics as `dotnet-publish-image` by using the `/p:PublishProfile=OpenShiftContainer` and setting `/p:OpenShiftImageName=<IMAGE_NAME>`.

### Adding the task to your OpenShift namespace

```
oc apply -f https://raw.githubusercontent.com/tmds/redhat-dotnet-container/main/tekton/dotnet-sdk.yaml
```
