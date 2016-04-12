# Creates pseudo distributed hadoop 2.6.0 on Ubuntu 14.04
#
# docker build -t lcdm/hadoop-info490 .

FROM sequenceiq/pam:ubuntu-14.04
MAINTAINER Edward J Kim <edward.junhyung.kim@gmail.com>

USER root

# Install all OS dependencies for fully functional notebook server
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && \
    apt-get install -yq --no-install-recommends \
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
    curl \
    tar \
    sudo \
    openssh-server \
    openssh-client \
    rsync \
    && apt-get clean
RUN locale-gen en_US.UTF-8 && \
    dpkg-reconfigure locales

# Configure environment
ENV SHELL /bin/bash
ENV NB_USER data_scientist
ENV NB_UID 1000
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# Create data_scientist user with UID=1000 and in the 'hadoop' group
RUN addgroup hadoop && \
    useradd -m -s /bin/bash -N -u $NB_UID -G hadoop $NB_USER

# Install Tini
RUN wget --quiet https://github.com/krallin/tini/releases/download/v0.9.0/tini && \
    echo "faafbfb5b079303691a939a747d7f60591f2143164093727e870b289a44d9872 *tini" | sha256sum -c - && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

# Configure environment
ENV CONDA_DIR /opt/conda
ENV PATH $CONDA_DIR/bin:$PATH

RUN mkdir -p /opt/conda && \
    chown $NB_USER:hadoop /opt/conda

USER $NB_USER

# Setup data_scientist home directory
RUN mkdir /home/$NB_USER/.jupyter && \
    mkdir /home/$NB_USER/.local

# Install conda as data_scientist
RUN cd /tmp && \
    mkdir -p $CONDA_DIR && \
    wget --quiet --no-check-certificate https://repo.continuum.io/miniconda/Miniconda3-3.19.0-Linux-x86_64.sh && \
    echo "9ea57c0fdf481acf89d816184f969b04bc44dea27b258c4e86b1e3a25ff26aa0 *Miniconda3-3.19.0-Linux-x86_64.sh" | sha256sum -c - && \
    /bin/bash Miniconda3-3.19.0-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-3.19.0-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda install --quiet --yes conda==3.19.1 && \
    conda clean -tipsy

# Install Python 3 packages as data_scientist
RUN conda install --yes \
    'conda=3.19.1' \
    'notebook=4.1*' \
    terminado  \
    'nose=1.3*' \
    'ipywidgets=4.1*' \
#    'pandas=0.17*' \
#    'matplotlib=1.5*' \
#    'scipy=0.17*' \
#    'seaborn=0.7*' \
#    'scikit-learn=0.17*' \
    && conda clean -tipsy

USER root

RUN mkdir -p /usr/java/default && \
    curl -Ls 'http://download.oracle.com/otn-pub/java/jdk/8u77-b03/jdk-8u77-linux-x64.tar.gz' -H 'Cookie: oraclelicense=accept-securebackup-cookie' | \
    tar --strip-components=1 -xz -C /usr/java/default/

ENV JAVA_HOME /usr/java/default/
ENV PATH $PATH:$JAVA_HOME/bin

# hadoop
RUN curl -s http://www-us.apache.org/dist/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.tar.gz | tar -xz -C /usr/local/
RUN cd /usr/local && ln -s ./hadoop-2.7.2 hadoop

ENV HADOOP_PREFIX /usr/local/hadoop
RUN chown -R $NB_USER:hadoop /usr/local/hadoop-2.7.2

# pig
RUN curl -s http://www-us.apache.org/dist/pig/pig-0.15.0/pig-0.15.0.tar.gz | tar -xz -C /usr/local/ && \
    chown -R $NB_USER:hadoop /usr/local/pig-0.15.0

ENV PATH $PATH:/usr/local/pig-0.15.0/bin

# passwordless ssh
RUN rm -f /etc/ssh/ssh_host_dsa_key /etc/ssh/ssh_host_rsa_key && \
    chown -R $NB_USER:hadoop /etc/ssh && \
    mkdir /home/$NB_USER/.ssh && \
    chown -R $NB_USER:hadoop /home/$NB_USER/.ssh

ADD ssh_config /home/$NB_USER/.ssh/config
RUN chmod 600 /home/$NB_USER/.ssh/config && \
    chown $NB_USER:hadoop /home/$NB_USER/.ssh/config

ADD bootstrap.sh /etc/bootstrap.sh
RUN chown $NB_USER:hadoop /etc/bootstrap.sh && \
    chmod 700 /etc/bootstrap.sh

ENV BOOTSTRAP /etc/bootstrap.sh

USER $NB_USER

RUN ssh-keygen -q -N '' -t dsa -f /etc/ssh/ssh_host_dsa_key && \
    ssh-keygen -q -N '' -t rsa -f /etc/ssh/ssh_host_rsa_key && \
    ssh-keygen -q -N '' -t rsa -f /home/$NB_USER/.ssh/id_rsa && \
    cat /home/$NB_USER/.ssh/id_rsa.pub >> /home/$NB_USER/.ssh/authorized_keys

RUN echo "export JAVA_HOME=$JAVA_HOME" >> /home/$NB_USER/.bashrc && \
    echo "export HADOOP_HOME=$HADOOP_PREFIX" >> /home/$NB_USER/.bashrc && \
    echo "export PATH=$PATH" >> /home/$NB_USER/.bashrc && \
    echo "export HADOOP_MAPRED_HOME=$HADOOP_PREFIX" >> /home/$NB_USER/.bashrc && \
    echo "export HADOOP_COMMON_HOME=$HADOOP_PREFIX" >> /home/$NB_USER/.bashrc && \
    echo "export HADOOP_HDFS_HOME=$HADOOP_PREFIX" >> /home/$NB_USER/.bashrc && \
    echo "export YARN_HOME=$HADOOP_PREFIX" >> /home/$NB_USER/.bashrc && \
    echo "export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_PREFIX/lib/native"  >> /home/$NB_USER/.bashrc && \
    echo "export HADOOP_CONF_DIR=$HADOOP_PREFIX/etc/hadoop" >> /home/$NB_USER/.bashrc && \
    echo "export HADOOP_YARN_HOME=$YARN_HOME" >> /home/$NB_USER/.bashrc

RUN sed -i '/^export JAVA_HOME/ s:.*:export JAVA_HOME=/usr/java/default\nexport HADOOP_PREFIX=/usr/local/hadoop\nexport HADOOP_HOME=/usr/local/hadoop\n:' $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh
RUN sed -i '/^export HADOOP_CONF_DIR/ s:.*:export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop/:' $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh
RUN . $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh

RUN mkdir $HADOOP_PREFIX/input && \
    cp $HADOOP_PREFIX/etc/hadoop/*.xml $HADOOP_PREFIX/input

# pseudo distributed
ADD core-site.xml.template $HADOOP_PREFIX/etc/hadoop/core-site.xml.template
RUN sed s/HOSTNAME/localhost/ /usr/local/hadoop/etc/hadoop/core-site.xml.template > /usr/local/hadoop/etc/hadoop/core-site.xml
ADD hdfs-site.xml $HADOOP_PREFIX/etc/hadoop/hdfs-site.xml

ADD mapred-site.xml.template $HADOOP_PREFIX/etc/hadoop/mapred-site.xml.template
RUN sed s/HOSTNAME/localhost/ /usr/local/hadoop/etc/hadoop/mapred-site.xml.template > /usr/local/hadoop/etc/hadoop/mapred-site.xml

ADD yarn-site.xml.template $HADOOP_PREFIX/etc/hadoop/yarn-site.xml.template
RUN sed s/HOSTNAME/localhost/ /usr/local/hadoop/etc/hadoop/yarn-site.xml.template > /usr/local/hadoop/etc/hadoop/yarn-site.xml

RUN $HADOOP_PREFIX/bin/hdfs namenode -format

# download native support
#RUN rm  /usr/local/hadoop/lib/native/*
#RUN curl -Ls http://dl.bintray.com/sequenceiq/sequenceiq-bin/hadoop-native-64-2.7.0.tar|tar -x -C /usr/local/hadoop/lib/native/
RUN mkdir -p /tmp/native
RUN curl -L https://github.com/sequenceiq/docker-hadoop-build/releases/download/v2.7.1/hadoop-native-64-2.7.1.tgz | tar -xz -C /tmp/native
RUN rm -rf /usr/local/hadoop/lib/native
RUN mv /tmp/native /usr/local/hadoop/lib

# workingaround docker.io build error
RUN ls -la /usr/local/hadoop/etc/hadoop/*-env.sh
RUN chmod +x /usr/local/hadoop/etc/hadoop/*-env.sh
RUN ls -la /usr/local/hadoop/etc/hadoop/*-env.sh

# fix the 254 error code
RUN sed  -i "/^[^#]*UsePAM/ s/.*/#&/"  /etc/ssh/sshd_config
RUN echo "UsePAM no" >> /etc/ssh/sshd_config
RUN echo "Port 2122" >> /etc/ssh/sshd_config

USER root
RUN service ssh start

USER $NB_USER
ENV LD_LIBRARY_PATH /usr/local/hadoop/lib/native/:$LD_LIBRARY_PATH

RUN service ssh start && \
    $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh && \
    $HADOOP_PREFIX/sbin/start-dfs.sh && \
    $HADOOP_PREFIX/bin/hdfs dfs -mkdir -p /user/$NB_USER

#RUN service ssh start && \
#    $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh && \
#    $HADOOP_PREFIX/sbin/start-dfs.sh && \
#    $HADOOP_PREFIX/bin/hdfs dfs -put $HADOOP_PREFIX/etc/hadoop/ input


# Configure container startup as root
USER root
# open ports for hadoop
EXPOSE 50020 50090 50070 50010 50075 8031 8032 8033 8040 8042 49707 22 8088 8030
# open ports for jupyterhub
EXPOSE 8888
WORKDIR /home/$NB_USER
ENTRYPOINT ["tini", "--"]
CMD ["/etc/bootstrap.sh"]

# Add local files as late as possible to avoid cache busting
COPY jupyter_notebook_config.py /home/$NB_USER/.jupyter/
RUN chown -R $NB_USER:hadoop /home/$NB_USER/.jupyter

USER $NB_USER
