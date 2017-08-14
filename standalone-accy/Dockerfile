FROM ubuntu:14.04

MAINTAINER Edward J Kim <edward.junhyung.kim@gmail.com>

USER root

# Install all OS dependencies for fully functional notebook server
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -yq --no-install-recommends \
    git \
    vim \
    wget \
    build-essential \
    python-dev \
    ca-certificates \
    bzip2 \
    unzip \
    libsm6 \
    sudo \
    locales \
    libxrender1 \
    libglib2.0-0 \
    libxext6 \
    libopenblas-base \
    libltdl7 \
    less \
    file \
    man-db \
    bsdmainutils \
    traceroute \
    && apt-get clean
RUN locale-gen en_US.UTF-8 && \
    dpkg-reconfigure locales

# Install Tini
RUN wget --quiet https://github.com/krallin/tini/releases/download/v0.10.0/tini && \
    echo "1361527f39190a7338a0b434bd8c88ff7233ce7b9a4876f3315c22fce7eca1b0 *tini" | sha256sum -c - && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

# Configure environment
ENV CONDA_DIR /opt/conda
ENV PATH $CONDA_DIR/bin:$PATH
ENV SHELL /bin/bash
ENV NB_USER data_scientist
ENV NB_UID 1000
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# Create data_scientist user with UID=1000 and in the 'users' group
RUN useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    mkdir -p $CONDA_DIR && \
    chown $NB_USER $CONDA_DIR

USER $NB_USER

# Setup data_scientist home directory
RUN mkdir /home/$NB_USER/.jupyter && \
    mkdir /home/$NB_USER/.local && \
    mkdir -p -m 700 /home/$NB_USER/.local/share/jupyter && \
    echo "cacert=/etc/ssl/certs/ca-certificates.crt" > /home/$NB_USER/.curlrc

# Install conda as data_scientist
RUN cd /tmp && \
    mkdir -p $CONDA_DIR && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-4.3.21-Linux-x86_64.sh && \
    /bin/bash Miniconda3-4.3.21-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-4.3.21-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda install --quiet --yes conda==4.3.21 && \
    $CONDA_DIR/bin/conda config --system --add channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    conda clean -tipsy

# Install Python 3 packages as data_scientist
RUN conda install --yes \
    'notebook=5.0.0' \
    'jupyterhub=0.7.2' \
    'terminado=0.6' \
    'nose=1.3.7' \
    'ipywidgets=6.0.0' \
    'pandas=0.19.2' \
    'matplotlib=2.0.0' \
    'scipy=0.18.1' \
    'seaborn=0.7.1' \
    'scikit-learn=0.18' \
    'scikit-image=0.12.3' \
    'sympy=1.0' \
    'cython=0.25.2' \
    'patsy=0.4.1' \
    'statsmodels=0.8.0' \
    'cloudpickle=0.2.2' \
    'dill=0.2.6' \
    'numba=0.31.0' \
    'bokeh=0.12.4' \
    'h5py=2.6.0' \
    'basemap=1.0.8' \
    'beautifulsoup4=4.5.3' \
    'gensim=0.13.4.1' \
    'lxml=3.7.3' \
    'networkx=1.11' \
    'nltk=3.2.2' \
    'snowballstemmer=1.2.1' \
    'sqlalchemy=1.1.5' \
    'joblib=0.11' \
    'tensorflow=1.0.1' \
    'theano=0.9.0' \
    'pymc3=3.0*' \
    'tweepy=3.5.0' \
    'keras=2.0.2' \
    'py2neo=4.0' \
    'tqdm=4.11.2' \
    && conda remove --quiet --yes --force qt pyqt \
    && conda clean -yt

# Activate ipywidgets extension in the environment that runs the notebook server
RUN jupyter nbextension enable --py widgetsnbextension --sys-prefix

# Configure ipython kernel to use matplotlib inline backend by default
RUN mkdir -p $HOME/.ipython/profile_default/startup
COPY mplimporthook.py $HOME/.ipython/profile_default/startup/

RUN pip install pip --upgrade && \
    pip install wordcloud==1.3.1 pydot-ng==1.0.0

# Configure container startup as root
USER root

RUN pip install nbgrader==0.5.2 && \
    nbgrader extension install

EXPOSE 8888
WORKDIR /home/$NB_USER
ENTRYPOINT ["tini", "--"]
CMD ["start-notebook.sh"]

# Add local files as late as possible to avoid cache busting
COPY start-notebook.sh /usr/local/bin/
COPY start-singleuser.sh /usr/local/bin/
COPY jupyter_notebook_config.py /home/$NB_USER/.jupyter/
RUN chown -R $NB_USER:users /home/$NB_USER/.jupyter

USER $NB_USER
