FROM frolvlad/alpine-glibc:alpine-3.10

ARG CONDA_VERSION="4.7.12.1"
ARG CONDA_MD5="81c773ff87af5cfac79ab862942ab6b3"
ARG CONDA_DIR="/opt/conda"

ENV PATH="$CONDA_DIR/bin:$PATH"
ENV PYTHONDONTWRITEBYTECODE=1

ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"
ENV HOME /home/${NB_USER}
ENV JUPYTER_ENABLE_LAB yes

USER root

# Install conda
RUN echo "**** install dev packages ****" && \
    apk add --no-cache --virtual .build-dependencies bash ca-certificates wget && \
    \
    echo "**** get Miniconda ****" && \
    mkdir -p "$CONDA_DIR" && \
    wget "http://repo.continuum.io/miniconda/Miniconda3-${CONDA_VERSION}-Linux-x86_64.sh" -O miniconda.sh && \
    echo "$CONDA_MD5  miniconda.sh" | md5sum -c && \
    \
    echo "**** install Miniconda ****" && \
    bash miniconda.sh -f -b -p "$CONDA_DIR" && \
    echo "export PATH=$CONDA_DIR/bin:\$PATH" > /etc/profile.d/conda.sh && \
    \
    echo "**** setup Miniconda ****" && \
    conda update --all --yes && \
    conda config --set auto_update_conda False && \
    conda install --yes -c conda-forge "nodejs=10.8" ipywidgets ipython-sql pandas=0.25* \
      python-dotenv nbstripout papermill pywinauto chardet watchdog Faker ipywidgets scipy=1.3* sqlalchemy=1.3* xlrd \
      pyppeteer jupyterlab=1.2.3 voila jupyterlab-git openssh \
    \
    echo "**** cleanup ****" && \
    apk del --purge .build-dependencies && \
    rm -f miniconda.sh && \
    conda clean --all --force-pkgs-dirs --yes && \
    find "$CONDA_DIR" -follow -type f \( -iname '*.a' -o -iname '*.pyc' -o -iname '*.js.map' \) -delete && \
    \
    echo "**** finalize ****" && \
    mkdir -p "$CONDA_DIR/locks" && \
    chmod 777 "$CONDA_DIR/locks"


USER $NB_UID
RUN conda init

RUN pip install nbinteract

# Lab Extensions
ENV NODE_OPTIONS="--max-old-space-size=4096"
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager@1.1 --no-build && \
  jupyter labextension install @jupyterlab/celltags --no-build && \ 
  jupyter labextension install @jupyterlab/toc --no-build && \
  jupyter labextension install @jupyter-voila/jupyterlab-preview  --no-build
  # this is not working..? I can't seem to get it to fail when I run it manually inside the above layer.
  # build and clean assets, from: https://github.com/jupyterlab/jupyterlab/issues/4930#issuecomment-446597498
RUN jupyter lab build && jupyter lab clean && \
  jlpm cache clean && \
  npm cache clean --force && \
  rm -rf $HOME/.node-gyp && \
  rm -rf $HOME/.local 
  
# Set up git extension to work
RUN jupyter serverextension enable --py jupyterlab_git

# Set notebook widgets to work
RUN jupyter nbextension enable --py widgetsnbextension

# Generate base jupyter config
RUN jupyter notebook --generate-config -y

# set up nbstripout
RUN mkdir -p /home/$NB_USER/.config/git 
RUN git init && nbstripout --global --attributes=/home/$NB_USER/.config/git/attributes --install && rm .git -rf

# set up for binder, aka copy in repo
COPY . ${HOME}/binder

# fix permissions on folders so end user can do as they wish
RUN fix-permissions $CONDA_DIR && \
  fix-permissions /home/$NB_USER
