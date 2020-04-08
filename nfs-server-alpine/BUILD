load("@rules_pkg//:pkg.bzl", "pkg_tar")
load("@io_bazel_rules_docker//container:container.bzl", "container_bundle", "container_image", "container_push")
load("@io_bazel_rules_docker//contrib:push-all.bzl", push_all = "container_push")
load("@io_bazel_rules_docker//contrib:test.bzl", "container_test")
load(":image-name.bzl", "image_name")

architectures = [
    "amd",
    "arm",
]

# create tar file out of our exports file so that it is placed
# in the correct location in our container
pkg_tar(
    name = "exports-file",
    srcs = ["exports"],
    extension = "tar.gz",
    mode = "644",
    package_dir = "/etc",
    strip_prefix = ".",
)

# define image build, test, and publish targets for both architectures using
# list comprehensions
[container_image(
    name = a,
    base = "@nfs_server_" + a + "_base//image",
    tars = [":exports-file"],
) for a in architectures]

[container_test(
    name = a + "_test",
    configs = [":tests.yaml"],
    image = ":" + a,
) for a in architectures]

[container_push(
    name = "push_" + a + "_container",
    format = "Docker",
    image = ":" + a,
    registry = "index.docker.io",
    repository = "shimmerjs/nfs-alpine-server",
    tag = a,
) for a in architectures]

# create a container bundle containing both architectures
container_bundle(
    name = "bundle",
    images = {image_name(a): ":" + a for a in architectures},
)

# push our bundle of containers
push_all(
    name = "push_images",
    bundle = ":bundle",
    format = "Docker",
)
