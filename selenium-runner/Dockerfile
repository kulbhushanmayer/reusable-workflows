# docker/selenium-runner/Dockerfile
FROM debian:bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8 \
    PATH=/opt/venv/bin:$PATH

# create non-root user
ARG RUNNER_USER=gha
ARG RUNNER_UID=1000
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    wget \
    gnupg \
    python3 \
    python3-venv \
    python3-pip \
    fonts-liberation \
    libnss3 \
    libxss1 \
    libasound2 \
    libgtk-3-0 \
    libgbm1 \
  && rm -rf /var/lib/apt/lists/*

# Install Google Chrome (APT)
RUN wget -qO- https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome.gpg \
  && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main" \
        > /etc/apt/sources.list.d/google-chrome.list \
  && apt-get update && apt-get install -y --no-install-recommends google-chrome-stable \
  && rm -rf /var/lib/apt/lists/*

# Python venv and wheelhouse
RUN python3 -m venv /opt/venv \
  && /opt/venv/bin/pip install --upgrade pip wheel setuptools

# Copy your wheelhouse (optional) and requirements if building with internet; if building offline, build wheelhouse first
# Copy wheelhouse tar if you maintain one in build context; otherwise, pips will fetch during image build.
# COPY wheels.tar.gz /tmp/
# RUN if [ -f /tmp/wheels.tar.gz ]; then tar -xzf /tmp/wheels.tar.gz -C /tmp/wheels; /opt/venv/bin/pip install --no-index --find-links=/tmp/wheels -r /workspace/requirements-tests.txt; fi

# Create working dir & add minimal scripts
WORKDIR /workspace
COPY requirements-tests.txt /workspace/requirements.txt
# Install test deps into venv (this is done at build time; if builder has internet)
RUN /opt/venv/bin/pip install --no-cache-dir -r /workspace/requirements-tests.txt

# Create non-root user and switch to it
RUN useradd -m -u ${RUNNER_UID} ${RUNNER_USER}
USER ${RUNNER_USER}
ENV HOME=/home/${RUNNER_USER}

# Default entrypoint: run bash (so GitHub Actions steps can run inside)
ENTRYPOINT ["/bin/bash", "-lc"]
CMD ["exec bash"]
