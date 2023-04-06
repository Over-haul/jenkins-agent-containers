FROM public.ecr.aws/e5r9m0c5/jenkins-jdk:v1.0.4

# Ensure system is up to date and install tools
RUN echo 'deb http://archive.ubuntu.com/ubuntu/ jammy-proposed restricted main multiverse universe' > /etc/apt/sources.list.d/jammy-proposed.list && \
    add-apt-repository ppa:deadsnakes/ppa && \
    apt update && \
    apt install -y \
        git \
        python3.7 \
        python3.7-venv \
        python3.7-dev \
        python3-pip && \
    curl https://bootstrap.pypa.io/get-pip.py -o /tmp/get-pip.py && \
    python3.7 /tmp/get-pip.py && \
    hash -r && \
    python3.7 -m pip install pipenv && \
    python3.7 -m pip install poetry && \
    apt autoremove -yqq --purge && \
    apt clean all && \
    rm -rf \
      /var/lib/apt/lists/* \
      /var/log/apt/* \
      /var/log/alternatives.log \
      /var/log/bootstrap.log \
      /var/log/dpkg.log \
      /var/tmp/* \
      /tmp/*
