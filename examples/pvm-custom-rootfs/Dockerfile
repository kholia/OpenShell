# SPDX-FileCopyrightText: Copyright (c) 2025-2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# Development-only OCI rootfs for exercising the experimental VM/PVM driver.
# Install system packages at image-build time because OpenShell workloads run
# without root privileges and cannot elevate through sudo.
FROM ubuntu:24.04

COPY dev-packages.txt /tmp/dev-packages.txt
COPY local-debs/ /tmp/local-debs/

RUN set -eux; \
    apt-get update; \
    sed -e '/^[[:space:]]*#/d' -e '/^[[:space:]]*$/d' \
        /tmp/dev-packages.txt \
        | xargs apt-get install -y --no-install-recommends; \
    set -- /tmp/local-debs/*.deb; \
    if [ -e "$1" ]; then \
        apt-get install -y --no-install-recommends "$@"; \
    fi; \
    rm -rf /var/lib/apt/lists/* /tmp/dev-packages.txt /tmp/local-debs

RUN apt-get update && \
DEBIAN_FRONTEND=noninteractive apt-get install -y \
    openbox \
    xvfb \
    x11vnc \
    novnc \
    websockify \
    dbus-x11 \
    xauth \
    x11-utils \
    x11-xserver-utils \
    libgtk-3-0 \
    libnss3 \
    libnspr4 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libcups2 \
    libdrm2 \
    libgbm1 \
    libx11-xcb1 \
    libxcomposite1 \
    libxdamage1 \
    libxfixes3 \
    libxrandr2 \
    libxkbcommon0 \
    libpango-1.0-0 \
    libcairo2 \
    fonts-dejavu-core \
    fonts-liberation \
    ca-certificates \
    curl \
    procps \
    libasound2t64 \
    vim \
    strace \
    gdb \
    && rm -rf /var/lib/apt/lists/*

RUN chmod o+x /opt/google/chrome/google-chrome || true

# Docker and Podman use the OCI identity. The VM driver injects and selects its
# configured sandbox UID/GID instead, and continues to use /sandbox.
RUN groupadd --gid 1500 dev \
    && useradd --uid 1500 --gid dev --create-home dev \
    && install -d -o dev -g dev /sandbox

WORKDIR /sandbox
USER dev

# OpenShell replaces CMD with its supervisor. This is only a useful fallback
# when inspecting the image with a conventional container runtime.
CMD ["bash"]
