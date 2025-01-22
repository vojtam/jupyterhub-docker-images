# jupyterhub docker images

my custom images for [https://hub.cloud.e-infra.cz](https://hub.cloud.e-infra.cz)

Documentation: [https://docs.cerit.io/en/web-apps/jupyterhub](https://docs.cerit.io/en/web-apps/jupyterhub)

the minimalnb+ssh image offers on top of the jupyter/minimal-notebook:
- openssh-server
- tmux
- btop
- quarto
- lazygit
- build-essential (gcc, g++, make, etc.) - needed for torch.compile()
