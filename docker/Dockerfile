# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

FROM nvcr.io/nvidia/cuda:13.0.2-cudnn-devel-ubuntu24.04

# Copy uv from official image
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

# Install wget, zstd, and sudo
RUN apt-get update && apt-get install -y wget zstd sudo && rm -rf /var/lib/apt/lists/* && apt-get clean

# Download and install standalone Python 3.12
RUN wget -O /tmp/python.tar.zst https://github.com/astral-sh/python-build-standalone/releases/download/20251217/cpython-3.12.12+20251217-x86_64_v3-unknown-linux-gnu-pgo+lto-full.tar.zst
RUN cd /tmp && tar -I zstd -xf python.tar.zst
RUN cp -r /tmp/python/install/* /usr/local/
RUN rm -rf /tmp/python.tar.zst /tmp/python

# Create a non-root user for running JupyterLab with passwordless sudo
RUN useradd -m -s /bin/bash jupyter && \
    echo "jupyter ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set up working directory
WORKDIR /home/jupyter

# Create venv and install packages inside it
ENV VIRTUAL_ENV=/home/jupyter/.venv
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
RUN uv venv --python /usr/local/bin/python3 $VIRTUAL_ENV
RUN uv pip install --upgrade pip
RUN uv pip install \
    jupyterlab \
    ipywidgets \
    numpy \
    pandas \
    matplotlib \
    jupyter-resource-usage \
    jupyterlab-execute-time

# Copy configuration and shell files
COPY jupyter_server_config.json /home/jupyter/.jupyter/jupyter_server_config.json
COPY bashrc /home/jupyter/.bashrc
COPY profile /home/jupyter/.profile
RUN chown -R jupyter:jupyter /home/jupyter

# Expose JupyterLab port
EXPOSE 8888

# Switch to non-root user
USER jupyter

# Activate venv and run JupyterLab
CMD ["/home/jupyter/.venv/bin/jupyter", "lab", "--config=/home/jupyter/.jupyter/jupyter_server_config.json"]
