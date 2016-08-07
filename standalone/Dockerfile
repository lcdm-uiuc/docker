FROM debian:jessie

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
    && apt-get clean
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Install Tini
RUN wget --quiet https://github.com/krallin/tini/releases/download/v0.9.0/tini && \
    echo "faafbfb5b079303691a939a747d7f60591f2143164093727e870b289a44d9872 *tini" | sha256sum -c - && \
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
    mkdir -p -m 700 /home/$NB_USER/.local/share/jupyter && \
    echo "cacert=/etc/ssl/certs/ca-certificates.crt" > /home/$NB_USER/.curlrc

# Install conda as data_scientist
RUN cd /tmp && \
    mkdir -p $CONDA_DIR && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-3.19.0-Linux-x86_64.sh && \
    echo "9ea57c0fdf481acf89d816184f969b04bc44dea27b258c4e86b1e3a25ff26aa0 *Miniconda3-3.19.0-Linux-x86_64.sh" | sha256sum -c - && \
    /bin/bash Miniconda3-3.19.0-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-3.19.0-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda install --quiet --yes conda==3.19.1 && \
    $CONDA_DIR/bin/conda config --system --add channels conda-forge && \
    conda clean -tipsy

# Install Python 3 packages as data_scientist
RUN conda install --yes \
    'notebook=4.2*' \
    terminado \
    'nose=1.3*' \
    'ipywidgets=5.1*' \
    'pandas=0.18*' \
    'matplotlib=1.5*' \
    'scipy=0.17*' \
    'seaborn=0.7*' \
    'scikit-learn=0.17*' \
    'scikit-image=0.11*' \
    'sympy=1.0*' \
    'cython=0.23*' \
    'patsy=0.4*' \
    'statsmodels=0.6*' \
    'cloudpickle=0.1*' \
    'dill=0.2*' \
    'numba=0.23*' \
    'bokeh=0.11*' \
    'h5py=2.5*' \
    'basemap=1.0*' \
    'beautifulsoup4=4.4*' \
    'pytables=3.2*' \
    'graphviz=2.38*' \
    'gensim=0.12*' \
    'lxml=3.6' \
    'networkx=1.11*' \
    'nltk=3.2*' \
    'pymongo=3.0*' \
    'snowballstemmer=1.2*' \
    'sqlalchemy=1.0*' \
    'sqlite=3.13*' \
    'joblib=0.9*' \
    && conda remove --quiet --yes --force qt pyqt \
    && conda clean -yt

# Install JupyterHub to get the jupyterhub-singleuser startup script
RUN pip --no-cache-dir install 'jupyterhub==0.5'

# Activate ipywidgets extension in the environment that runs the notebook server
RUN jupyter nbextension enable --py widgetsnbextension --sys-prefix

# Configure ipython kernel to use matplotlib inline backend by default
RUN mkdir -p $HOME/.ipython/profile_default/startup
COPY mplimporthook.py $HOME/.ipython/profile_default/startup/

RUN pip install --no-deps git+git://github.com/Theano/Theano.git && \
    pip install --no-deps git+git://github.com/pymc-devs/pymc3.git && \
    pip install tweepy wordcloud cassandra-driver py2neo==3b1 keras pydot-ng

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
