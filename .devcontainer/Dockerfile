# Build arguments for specific commits of the stable-diffusion-webui
ARG SD_WEBUI_COMMIT="1937682a20f7f0442311a1ede68f9f0cb480163b"

# Define environment variables for directory paths
ARG SD_WEBUI_DIR=/opt/stable-diffusion-webui
ARG SD_VENV_DIR="${SD_WEBUI_DIR}/.venv"

ARG PYTHON_VER=3.11
ARG PYTHON_BIN="/usr/bin/python${PYTHON_VER}"

ARG USER_UID=1000
ARG USER_GID=1000

# =============== Install necessary system dependencies (base) stage ===============

FROM mcr.microsoft.com/devcontainers/base:ubuntu24.04 AS base

ARG PYTHON_VER

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      libglib2.0-0 \
      libgl1 \
      libtcmalloc-minimal4t64 \
      git \
      curl \
      ca-certificates \
      bc \
      software-properties-common && \
    add-apt-repository -y ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
      python${PYTHON_VER} \
      python${PYTHON_VER}-venv && \
    rm -rf /etc/apt/sources.list.d/deadsnakes-ubuntu-ppa-*.list && \
    apt-get purge -y software-properties-common && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# =============== Python venv preparation stage ===============

FROM base AS venv

ARG SD_VENV_DIR
ARG SD_WEBUI_COMMIT
ARG PYTHON_BIN
ARG USER_UID
ARG USER_GID

ARG UV_INSTALL_DIR="/tmp/bin"
RUN curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR=${UV_INSTALL_DIR} sh && \
    install -d -o ${USER_UID} -g ${USER_GID} ${SD_VENV_DIR}
USER ${USER_UID}:${USER_GID}
RUN "${PYTHON_BIN}" -m venv "${SD_VENV_DIR}" && \
    curl https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui/${SD_WEBUI_COMMIT}/requirements_versions.txt -o /tmp/requirements_versions.txt && \
    "${UV_INSTALL_DIR}/uv" pip install --python "${SD_VENV_DIR}/bin/python" --no-cache-dir --link-mode=copy -r /tmp/requirements_versions.txt

# =============== Application assembly stage ===============

FROM base

ARG SD_VENV_DIR
ARG SD_WEBUI_DIR
ARG SD_WEBUI_COMMIT
ARG USER_UID
ARG USER_GID

# Clone stable-diffusion-webui
# Do not remove .git as the webui will attempt to clone it on startup if it's missing.
RUN install -d -o ${USER_UID} -g ${USER_GID} "${SD_WEBUI_DIR}"
USER ${USER_UID}:${USER_GID}
RUN git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui "${SD_WEBUI_DIR}" && \
    git -C "${SD_WEBUI_DIR}" checkout "${SD_WEBUI_COMMIT}"

COPY --from=venv --chown=${USER_UID}:${USER_GID} "${SD_VENV_DIR}" "${SD_VENV_DIR}"

# CMD and ENTRYPOINT are not necessarily required for the container to run.
