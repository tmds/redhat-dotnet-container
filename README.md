# redhat-dotnet-container

This repo contains prototypes for improving the UX when using Red Hat .NET container images with the .NET SDK.

## RedHat.Container package

Adding this NuGet package makes the SDK use Red Hat .NET base images instead of Microsoft .NET base images. To opt-out of this behavior, you can set `UseRedHatDotnetImage` to `false`.
The image locations can be overridden by setting `RedHatAspNet60RuntimeImage`, `RedHatAspNet70RuntimeImage`, ... .

When the container image is built on a Kubernetes cluster, the package defaults to use a `dotnet` ImageStream image from the internal OpenShift image registry (even when `UseRedHatDotnetImage` is `false`). To opt-out of this behavior, you can set `UseOpenShiftDotnetImage` to `false`.
By default, the package looks for the dotnet images in the build namespace. Setting `OpenShiftDotnetNamespace` allows to specify the namespace.

To facilicate publishing to the OpenShift internal image registry, when no `ContainerRegistry` is set, the image is published to the OpenShift internal image registry. By default, the build namespace is used. This can be overridden by setting `OpenShiftNamespace`. Like with the .NET SDK, the image name can be controlled by setting `ContainerRepository`.

Overview of properties:

| Property                   | Description | Default value |
|----------------------------|-------------|---------------|
| UseRedHatDotnetImage       | Use Red Hat .NET images.                   | true |
| UseOpenShiftDotnetImage    | On a cluster, use .NET imagestream images. | true |
| OpenShiftDotnetNamespace   | Internal image registry namespace for `UseOpenShiftDotnetImage` images. | _current namespace_ |
| OpenShiftNamespace         | Internal image registry namespace to push to.                           | _current namespace_ |
| RedHatAspNet60RuntimeImage | Location of the .NET 6 ASP.NET Core runtime image. | registry.access.redhat.com/ubi8/dotnet-60-runtime:latest |
| RedHatAspNet70RuntimeImage | Location of the .NET 7 ASP.NET Core runtime image. | registry.access.redhat.com/ubi8/dotnet-70-runtime:latest |
| RedHatAspNet80RuntimeImage | Location of the .NET 8 ASP.NET Core runtime image. | registry.access.redhat.com/ubi8/dotnet-80-runtime:latest |
