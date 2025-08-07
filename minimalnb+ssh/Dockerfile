FROM jupyter/minimal-notebook:hub-4.0.2 as base

USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    software-properties-common \
    ca-certificates \
    gnupg && \
    add-apt-repository --yes ppa:ultradvorka/ppa && \
    apt-get update && \
    apt-get -y dist-upgrade && \
    apt-get install -y --no-install-recommends \
    fakeroot \
    dumb-init \
    vim \
    build-essential \
    graphviz \
    tmux \
    curl \
    less \
    htop \
    hstr \
    btop \
    nano \
    patch \
    neovim \
    openssh-server \
    fonts-dejavu \
    unixodbc \
    unixodbc-dev \
    r-cran-rodbc \
    gfortran \
    gcc && \
    # Clean up APT caches
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

#### Install Rust (for vLLM) #### 
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y && \
    rm -rf /root/.cargo/registry/cache && \
    rm -rf /root/.cargo/git/db

RUN curl -sS https://starship.rs/install.sh | sh -s -- --yes

#### Install zellij ####
RUN cargo install --locked zellij


# Ensure Rust is available in PATH
ENV PATH="/root/.cargo/bin:$PATH"
#### Install Rust end ####

#### Install uv ####
# Download the latest uv installer
ADD https://astral.sh/uv/install.sh /uv-installer.sh

# Run the installer then remove it
RUN sh /uv-installer.sh && rm /uv-installer.sh

# Ensure the installed binary is on the `PATH`
ENV PATH="/root/.local/bin/:$PATH"
#### Install uv end ####

# R packages including IRKernel which gets installed globally.
# r-e1071: dependency of the caret R package
RUN mamba install --yes \
    'r-base' \
    'r-caret' \
    'r-crayon' \
    'r-devtools' \
    'r-e1071' \
    'r-forecast' \
    'r-hexbin' \
    'r-htmltools' \
    'r-htmlwidgets' \
    'r-irkernel' \
    'r-nycflights13' \
    'r-randomforest' \
    'r-rcurl' \
    'r-glue' \
    'r-rmarkdown' \
    'r-reticulate' \
    'r-rodbc' \
    'r-rsqlite' \
    'r-shiny' \
    'r-tidymodels' \
    'r-tidyverse' \
    'unixodbc' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"


# Install LazyGit
RUN LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | \
    grep -Po '"tag_name": *"v\K[^"]*') && \
    curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/download/v${LAZYGIT_VERSION}/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz" && \
    tar xf lazygit.tar.gz lazygit && \
    install lazygit -D -t /usr/local/bin/ && \
    rm -f lazygit.tar.gz lazygit


# Install Quarto
RUN QUARTO_DEB_URL="https://github.com/quarto-dev/quarto-cli/releases/download/v1.7.13/quarto-1.7.13-linux-amd64.deb" && \
    curl -Lo quarto.deb $QUARTO_DEB_URL && \
    dpkg -i quarto.deb || apt-get install -f -y && \
    rm -f quarto.deb

RUN curl -LsSf https://astral.sh/uv/install.sh | sh
# Make sudo working
RUN rm -f /usr/bin/sudo; ln -sr /usr/bin/fakeroot /usr/bin/sudo
# Set default shell to bash
RUN rm -f /bin/sh && ln -sr /bin/bash /bin/sh



# Important, clean host keys
RUN rm -f /etc/ssh/ssh_host_*
# Add ssh configuration
COPY --chown=1000:100 sshd.conf /etc/ssh/sshd_config.d/sshd.conf
# Replace start script so that sshd stars as well
COPY --chown=1000:100 start-notebook.sh /usr/local/bin/start-notebook.sh

RUN conda install --quiet --yes nb_conda_kernels

# Create new image from scratch
FROM scratch

#RUN curl 

# kaniko runner compatible, for docker, it is sufficent to replace all 
# the copy commands by single one: COPY --chown=1000:100 --from=base / / 
COPY --chown=1000:100 --from=base  /bin /bin
COPY --chown=1000:100 --from=base  /sbin /sbin
COPY --chown=1000:100 --from=base  /lib64 /lib64
COPY --chown=1000:100 --from=base  /libx32 /libx32
COPY --chown=1000:100 --from=base  /lib /lib
COPY --chown=1000:100 --from=base  /boot /boot
COPY --chown=1000:100 --from=base  /etc /etc
COPY --chown=1000:100 --from=base  /home /home
COPY --chown=1000:100 --from=base  /opt /opt
COPY --chown=1000:100 --from=base  /root /root
COPY --chown=1000:100 --from=base  /tmp /tmp
COPY --chown=1000:100 --from=base  /usr /usr
COPY --chown=1000:100 --from=base  /var /var

USER 1000

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

ENV PATH=$PATH:/opt/conda/bin
ENV LD_LIBRARY_PATH=/opt/conda/lib

WORKDIR /home/jovyan

# Install Starship prompt   
RUN echo eval "$(starship init bash)" >> ~/.bashrc

# Required for GPU
RUN mkdir -p /usr/share/glvnd/egl_vendor.d /usr/share/egl/egl_external_platform.d /etc/vulkan/icd.d /etc/vulkan/implicit_layer.d

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV XDG_CACHE_HOME="/home/${NB_USER}/.cache/"
ENV XLA_FLAGS=--xla_gpu_cuda_data_dir=/opt/conda/lib
ENV PS1 '\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
ENV PS2 '> '
ENV PS4 '+ '
ENTRYPOINT ["dumb-init"]
CMD ["start-notebook.sh"]
