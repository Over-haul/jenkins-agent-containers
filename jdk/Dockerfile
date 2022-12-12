FROM ubuntu:jammy-20221130

ENV DEBIAN_FRONTEND=noninteractive
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
ENV TERM=linux
ARG JAVA_MAJOR_VERSION=11
ENV MAVEN_HOME=/usr/share/maven
ARG MAVEN_VERSION=3.8.1
ARG MAVEN_SHA=0ec48eb515d93f8515d4abe465570dfded6fa13a3ceb9aab8031428442d9912ec20f066b2afbf56964ffe1ceb56f80321b50db73cf77a0e2445ad0211fb8e38d
ARG MAVEN_BASE_URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries
ENV GRADLE_HOME=/opt/gradle
ARG GRADLE_VERSION=6.8.3
ARG GRADLE_DOWNLOAD_SHA256=7faa7198769f872826c8ef4f1450f839ec27f0b4d5d1e51bade63667cbccd205
ENV user=jenkins
ENV group=jenkins
ENV uid=1000
ENV gid=1000
ENV AGENT_WORKDIR=/home/${user}/agent
ENV DOCKER_VERSION=20.10.5
ENV DOCKER_COMPOSE_VERSION=1.29.2
ENV KUBECTL_VERSION=v1.22.0
ENV KOMPOSE_VERSION=v1.22.0

ADD 01_nodoc /etc/dpkg/dpkg.conf.d/01_nodoc
# Ensure system is up to date and install tools
COPY jenkins-agent /usr/local/bin/jenkins-agent
RUN echo '===> Starting setup' && \
    echo 'deb http://archive.ubuntu.com/ubuntu/ jammy-proposed restricted main multiverse universe' > /etc/apt/sources.list.d/jammy-proposed.list && \
    echo '===> Installing apt dependencies' && \
    apt update && \
    apt install -y \
        git \
        git-lfs \
        jq \
        python3-setuptools \
        iputils-ping \
        curl \
        unzip \
        wget \
        vim-common \
        "openjdk-${JAVA_MAJOR_VERSION}-dbg" \
        "openjdk-${JAVA_MAJOR_VERSION}-jdk-headless" \
        software-properties-common && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys CC86BB64 && \
    echo '===> Installing nodejs apt repository' && \
    curl -sL https://deb.nodesource.com/setup_14.x -o /tmp/nodesource_setup.sh && \
    bash /tmp/nodesource_setup.sh && \
    apt install -y nodejs && \
    echo "===> Installing Maven ${MAVEN_VERSION}" && \
    mkdir -p /usr/share/maven /usr/share/maven/ref && \
    curl -fsSL -o /tmp/apache-maven.tar.gz ${MAVEN_BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz && \
    echo "${MAVEN_SHA}  /tmp/apache-maven.tar.gz" | sha512sum --check - && \
    tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 && \
    rm -f /tmp/apache-maven.tar.gz && \
    ln -s /usr/share/maven/bin/mvn /usr/bin/mvn && \
    echo "===> Installing Gradle " && \
    curl -fsSL -o gradle.zip "https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip" && \
    echo "${GRADLE_DOWNLOAD_SHA256} *gradle.zip" | sha256sum --check - && \
    unzip gradle.zip && \
    rm gradle.zip && \
    mv "gradle-${GRADLE_VERSION}" "${GRADLE_HOME}/" && \
    ln --symbolic "${GRADLE_HOME}/bin/gradle" /usr/bin/gradle && \
    hash -r && \
    echo "===> Testing if maven and gradle work" && \
    mvn -v && \
    gradle --version && \
    echo '===> Make jenkins-agent executable' && \
    chmod +x /usr/local/bin/jenkins-agent && \
    ln -s /usr/local/bin/jenkins-agent /usr/local/bin/jenkins-slave && \
    echo '===> Creating jenkins group, user and home' && \
    groupadd -g ${gid} ${group} && \
    useradd -c "Jenkins user" -d /home/${user} -u ${uid} -g ${gid} -m ${user} && \
    mkdir /home/${user}/.jenkins && mkdir -p ${AGENT_WORKDIR} && \
    chown -R jenkins: /home/${user} && \
    echo '===> Installing python pip' && \
    curl -sL https://bootstrap.pypa.io/get-pip.py -o /tmp/get-pip.py && \
    python3.10 /tmp/get-pip.py && \
    echo '===> Installing pip dependencies' && \
    python3.10 -m pip install \
      boto3 \
      jira \
      prefect==1.2.0 \
      psycopg2-binary==2.9.3 \
      PyGithub \
      pytz \
      requests && \
    echo '===> Installing reviewdog' && \
    REVIEWDOG_VERSION="$(curl -s 'https://api.github.com/repos/reviewdog/reviewdog/releases/latest' | jq -r .tag_name | tr -d 'v')" && \
    curl -sL \
      "https://github.com/reviewdog/reviewdog/releases/download/v${REVIEWDOG_VERSION}/reviewdog_${REVIEWDOG_VERSION}_Linux_x86_64.tar.gz" \
      -o /tmp/reviewdog.tar.gz && \
    tar -xzvf /tmp/reviewdog.tar.gz -C /tmp/ && \
    mv /tmp/reviewdog /usr/local/bin/reviewdog && \
    chmod +x /usr/local/bin/reviewdog && \
    reviewdog --version && \
    echo '===> Installing yq' && \
    YQ_VERSION="$(curl -s 'https://api.github.com/repos/mikefarah/yq/releases/latest' | jq -r .tag_name)" && \
    curl -sL \
      "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64" \
      -o /usr/local/bin/yq && \
    chmod +x /usr/local/bin/yq && \
    yq --version && \
    echo '===> Installing kubectl' && \
    #KUBECTL_VERSION="$(curl -sL https://dl.k8s.io/release/stable.txt)" && \
    curl -sL \
      "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
      -o /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl && \
    kubectl --help && \
    echo '===> Installing kompose' && \
    #KOMPOSE_VERSION="$(curl -s 'https://api.github.com/repos/kubernetes/kompose/releases/latest' | jq -r .tag_name)" && \
    curl -sL \
      "https://github.com/kubernetes/kompose/releases/download/${KOMPOSE_VERSION}/kompose-linux-amd64" \
      -o /usr/local/bin/kompose && \
    chmod +x /usr/local/bin/kompose && \
    kompose version && \
    echo '===> Installing docker-compose' && \
    #DOCKER_COMPOSE_VERSION="$(curl -s 'https://api.github.com/repos/docker/compose/releases/latest' | jq -r .tag_name)" && \
    curl -sL \
      "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" \
      -o /usr/local/bin/docker-compose && \
    chmod +x /usr/local/bin/docker-compose && \
    docker-compose --version && \
    echo '===> Installing packer' && \
    PACKER_VERSION="$(curl -s 'https://api.github.com/repos/hashicorp/packer/releases/latest' | jq -r .tag_name | tr -d 'v')" && \
    curl -sL \
      "https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_linux_amd64.zip" \
      -o /tmp/packer.zip && \
    unzip /tmp/packer.zip -d /usr/local/bin/ && \
    chmod +x /usr/local/bin/packer && \
    packer --version && \
    echo '===> Installing terraform 0.12.31' && \
    curl -sL \
      "https://releases.hashicorp.com/terraform/0.12.31/terraform_0.12.31_linux_amd64.zip" \
      -o /tmp/terraform.zip && \
    unzip /tmp/terraform.zip -d /usr/local/bin/ && \
    chmod +x /usr/local/bin/terraform && \
    terraform --version && \
    echo '===> Installing jmeter and plugins' && \
    curl -sL 'https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.2.1.tgz' -o /opt/jmeter.tgz && \
    echo '====> untar jmeter.tgz' && \
    tar -xzvf /opt/jmeter.tgz -C /opt/ && \
    echo '====> untar moving jmeter to /opt/jmeter' && \
    mv -f /opt/apache-jmeter-5.2.1* /opt/jmeter && \
    echo '====> linking /opt/jmeter/bin/jmeter to /usr/bin/jmeter' && \
    ln -s /opt/jmeter/bin/jmeter /usr/bin/jmeter && \
    echo '====> downloading jmeter-plugins' && \
    curl -sL https://jmeter-plugins.org/files/packages/jpgc-casutg-2.9.zip -o /opt/jmeter-plugins.zip && \
    echo '====> unzip jmeter-plugins' && \
    unzip /opt/jmeter-plugins.zip -d /opt && cp -r /opt/lib /opt/jmeter/ && \
    echo '====> Removing jmeter.tgz and jmeter-plugins.zip' && \
    rm -rf /opt/lib /opt/jmeter.tgz /opt/jmeter-plugins.zip && \
    echo '===> Installing awscli v2' && \
    curl -sL 'https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip' -o /tmp/awscli-exe-linux-x86_64.zip && \
    unzip /tmp/awscli-exe-linux-x86_64.zip -d /tmp/ && \
    /tmp/aws/install --update && \
    /usr/local/bin/aws --version && \
    echo '===> Installing azure cli' && \
    curl -sL https://aka.ms/InstallAzureCLIDeb -o /tmp/azure-cli.sh && \
    bash /tmp/azure-cli.sh && \
    az version && \
    echo '===> Cleaning up' && \
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

ENTRYPOINT ["jenkins-slave"]
