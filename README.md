
# Docker Images for JupyterHub

This directory contains the images for running JupyterHub instances on [CERIT-SC](https://docs.cerit.io/en/docs/news) Kubernetes infrastructure.

The images are based on [CERIT-SC docker images repository](https://github.com/CERIT-SC/docker-images)

- `minimalnb+ssh` contains working `sshd` client that allows to connect to the running instance through `SSH` protocol.
additionally, it contains the following quality-of-life packages that I use for my ML / data science project needs:
- `Quarto` - authoring system for interactive reports
- `uv` - python package manager
- `hstr` - interactive bash history browser
- `tmux` - screen session manager
- `btop` - modern spin on the `(h)top` system resource dashboard.
- `graphviz` binary - for generating diagrams of neural networks
- `LazyGit`- for fast git workflow
- build-essential (gcc, g++, make, etc.) - needed for torch.compile()


The images are hosted at [CERIT Harbor](https://cerit.io/harbor/projects/240/repositories/sshminjupyter/artifacts-tab)