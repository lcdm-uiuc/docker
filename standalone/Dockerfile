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
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-4.2.12-Linux-x86_64.sh && \
    echo "c59b3dd3cad550ac7596e0d599b91e75d88826db132e4146030ef471bb434e9a *Miniconda3-4.2.12-Linux-x86_64.sh" | sha256sum -c - && \
    /bin/bash Miniconda3-4.2.12-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-4.2.12-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda config --system --add channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    conda clean -tipsy

# Install Python 3 packages as data_scientist
RUN conda create -n py36 python=3.6 \
    notebook \
#    jupyterhub=0.7 \
    terminado \
    nose \
    ipywidgets \
    pandas \
    matplotlib \
    scipy \
    seaborn \
    scikit-learn \
    scikit-image \
    sympy \
    cython \
    patsy \
    statsmodels \
    cloudpickle \
    dill \
    numba \
    bokeh \
    h5py \
    basemap \
    beautifulsoup4 \
    pytables \
    graphviz \
    gensim \
    lxml \
    networkx \
    nltk \
    pymongo \
    snowballstemmer \
    sqlalchemy \
    sqlite \
    joblib \
    jupyter \
    && conda clean -yt

ENV PATH $CONDA_DIR/envs/py36/bin:$PATH
RUN /bin/bash -c "source $CONDA_DIR/bin/activate py36"
    
RUN pip install --upgrade pip && \
    pip install --no-deps git+git://github.com/Theano/Theano.git && \
    pip install --no-deps git+git://github.com/pymc-devs/pymc3.git && \
    # cassandra-driver 3.7  has a bug, so downgrade to 3.6
    # https://datastax-oss.atlassian.net/browse/PYTHON-649
    pip install tweepy wordcloud cassandra-driver==3.6 py2neo==3b1 keras pydot-ng && \
    # PyMC3 depends on tqdm (for now) but remove in future versions
    # https://github.com/pymc-devs/pymc3/pull/1408
    pip install tqdm && \
    pip install jupyterhub tensorflow

# Activate ipywidgets extension in the environment that runs the notebook server
RUN jupyter nbextension enable --py widgetsnbextension --sys-prefix

# Configure ipython kernel to use matplotlib inline backend by default
RUN mkdir -p $HOME/.ipython/profile_default/startup
COPY mplimporthook.py $HOME/.ipython/profile_default/startup/


# Configure container startup as root
USER root

RUN pip install git+git://github.com/EdwardJKim/nbgrader.git && \
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
