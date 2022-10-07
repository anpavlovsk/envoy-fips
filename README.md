# Building Envoy in a FIPS compliant mode 

Envoy has support for building in a FIPS compliant mode. The upstream project does not distribute a FIPS compliant Envoy container image, but combining the documented process with the processes for building the Envoy executable and container image, we can produce one.

Again we will need the Envoy source code checked out to the version to build and Docker installed on your computer. The simplest way to build Envoy without having to learn Bazel and set up a C++ toolchain on your computer is to build using the Envoy build container image which contains the necessary tools pre-installed. Note that if you do build with FIPS mode outside of the build container, you can only do so on a Linux-amd64 architecture.

We can first compile the Envoy binary by running the following in a bash shell from the Envoy source directory:
````
ENVOY_DOCKER_BUILD_DIR=~/output ./ci/run_envoy_docker.sh 'BAZEL_BUILD_EXTRA_OPTIONS=--define=boringssl=fips ./ci/do_ci.sh bazel.release.server_only'
````
Once that build completes, you should have a file named envoy_binary.tar.gz in your specified output directory (~/output ).

If you would like to build an image with Envoy according to your own specifications, you can unpack the resulting tar archive and you will find a stripped Envoy binary in the build_release_stripped directory and a unstripped Envoy with debug info in the build_release directory.

To build an image matching the canonical Envoy upstream release image (envoyproxy/envoy), run the following:
````
# Make ./linux/amd64 directories.
mkdir -p ./linux/amd64
# Untar tar archive from build step.
tar xzf ~/output/envoy/envoy_binary.tar.gz -C ./linux/amd64
# Run the Docker image build.
docker build -f ci/Dockerfile-envoy -t envoy .
````
Once you have an image built, you can tag it as needed, push the image to a registry, and use it in an Envoy deployment.
Start container 
````
docker run -d --name envoy -p 9901:9901 -p 10000:10000 envoy:latest
````
The correctness of the resulting FIPS build can be verified by checking the presence of BoringSSL-FIPS in the --version output.
````
sudo docker run -ti --rm --entrypoint envoy envoy --version
````
Output

````
ubuntu@ip-172-31-7-3:~/envoy$ sudo docker run -ti --rm --entrypoint envoy envoy --version

envoy  version: 0a0fc44f86a94546d907a47253741478786394df/1.24.0-dev/Clean/RELEASE/BoringSSL-FIPS
````

