FROM jupyter/scipy-notebook:latest

USER root

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    graphviz \
    && rm -rf /var/lib/apt/lists/*

USER ${NB_UID}

# Install Python packages for the RLM tutorial
COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt

# Copy the tutorial notebook into the default work directory
COPY --chown=${NB_UID}:${NB_GID} notebooks/ /home/jovyan/work/

WORKDIR /home/jovyan/work
