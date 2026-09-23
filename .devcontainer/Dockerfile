# IvorySQL Build and Test Container - Modern Ubuntu
# Alternative to CentOS with same capabilities

FROM ubuntu:22.04

# Prevent interactive prompts
ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies matching the workflow requirements
RUN apt-get update && apt-get install -y \
    # Build tools
    build-essential git lcov bison flex pkg-config cppcheck \
    # For meson installation
    curl \
    # Ninja: Ubuntu 22.04 ships ninja-build 1.10.1 - the same upstream
    # version the workflows pin - natively for every architecture this
    # image supports, while the pinned ninja v1.10.1 release only
    # provides an x86_64 Linux binary (upstream added Linux arm64
    # assets only in v1.12.1)
    ninja-build \
    # Core dependencies
    libkrb5-dev libssl-dev libldap-dev libpam-dev \
    libxml2-dev libxslt-dev libreadline-dev libedit-dev \
    zlib1g-dev uuid-dev libossp-uuid-dev libuuid1 e2fsprogs \
    # ICU support
    libicu-dev \
    # Language support
    python3-dev python3-setuptools tcl-dev libperl-dev gettext \
    # Perl test modules
    libipc-run-perl libtime-hires-perl libtest-simple-perl \
    # LLVM/Clang
    llvm clang \
    # LZ4 compression
    liblz4-dev \
    # System libraries
    libselinux1-dev libsystemd-dev \
    # GSSAPI
    libgssapi-krb5-2 \
    # Locale support
    locales \
    # For dev containers
    sudo tini \
    # Debugging tools
    gdb \
    && rm -rf /var/lib/apt/lists/*

# Install meson from source (matching the workflow version). Ninja comes
# from the ninja-build distro package installed above.
RUN curl -L "https://github.com/mesonbuild/meson/releases/download/1.0.1/meson-1.0.1.tar.gz" -o /tmp/meson-1.0.1.tar.gz \
    && tar xzf /tmp/meson-1.0.1.tar.gz -C /tmp \
    && cd /tmp/meson-1.0.1 && python3 setup.py install \
    && rm -rf /tmp/meson-1.0.1*

# Set up locale
RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

# Create ivorysql user with matching host UID/GID (1000:1000)
# and grant sudo privileges without password
ARG USER_UID=1000
ARG USER_GID=1000
RUN groupadd -g ${USER_GID} ivorysql || true && \
    useradd -m -u ${USER_UID} -g ${USER_GID} -d /home/ivorysql -s /bin/bash ivorysql && \
    echo "ivorysql ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set working directory
WORKDIR /home/ivorysql/IvorySQL

# Switch to ivorysql user for builds
USER ivorysql

# Default command
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["/bin/bash"]
