# Use the latest Ubuntu LTS release
FROM ubuntu:22.04

# Set the environment variable to avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Update and install prerequisites for adding repositories
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    wget \
    gnupg \
    ca-certificates \
    lsb-release \
    software-properties-common \
    python3-dev \
    python3-pip

# Add LLVM repository and its GPG key
RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | \
    gpg --dearmor | \
    tee /usr/share/keyrings/llvm-archive-keyring.gpg >/dev/null && \
    echo "deb [signed-by=/usr/share/keyrings/llvm-archive-keyring.gpg] http://apt.llvm.org/$(lsb_release -cs)/ llvm-toolchain-$(lsb_release -cs)-17 main" | \
    tee /etc/apt/sources.list.d/llvm.list

# Update package lists after adding new repositories
RUN apt-get update

# Install the specified packages, excluding Meson
RUN apt-get install -y --no-install-recommends \
    bison \
    build-essential \
    busybox-static \
    cargo \
    clang-17 \
    cmake \
    coreutils \
    cpio \
    elfutils \
    file \
    flex \
    gcc \
    gcc-multilib \
    git \
    iproute2 \
    jq \
    kbd \
    kmod \
    libcap-dev \
    libelf-dev \
    libssl-dev \
    libunwind-dev \
    libvirt-clients \
    libzstd-dev \
    llvm-17 \
    linux-headers-generic \
    linux-tools-common \
    linux-tools-generic \
    make \
    ninja-build \
    pahole \
    pkg-config \
    python3-requests \
    qemu-kvm \
    rsync \
    rustc \
    udev \
    zstd

# Install the latest version of Meson using pip
RUN pip3 install --upgrade pip && \
    pip3 install meson

# Clean up APT when done
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Set the default command to bash
CMD ["/bin/bash"]
