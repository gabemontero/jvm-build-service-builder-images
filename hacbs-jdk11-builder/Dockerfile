FROM registry.access.redhat.com/ubi8/ubi-minimal
USER root
ENV \
    LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    LC_ALL="en_US.UTF-8" \
    APP_HOME="/home/jboss" \
    MAVEN_HOME="/usr/share/maven" \
    MAVEN_OPTS="-XX:+TieredCompilation -XX:TieredStopAtLevel=1" \
    MAVEN_VERSION="3.8.7" \
    GRADLE_OPTS="-Dorg.gradle.daemon=false" \
    GRADLE_MANIPULATOR_HOME="/usr/share/gradle-manipulator" \
    GRADLE_MANIPULATOR_VERSION="3.11" \
    HOME="/home/jboss" \
    NSS_WRAPPER_PASSWD="/etc/passwd" \
    CLI_JAR_DOWNLOAD_SHA256=cbd2cf43ea34953f9a98f85639a4a25d2fd8827dcb9f1039dc7c48233698ada5 \
    ANALYZER_INIT_DOWNLOAD_SHA256=7d1be22c516e24a67d8c6f3a1e7f949a5d5d8245e918458cae1e20c4226a1acf \
    PATH="$PATH:$JAVA_HOME/bin" \
    BASE_URL=https://dlcdn.apache.org/maven/maven-3/3.8.7/binaries \
    MAVEN_SHA=21c2be0a180a326353e8f6d12289f74bc7cd53080305f05358936f3a1b6dd4d91203f4cc799e81761cf5c53c5bbe9dcc13bdb27ec8f57ecf21b2f9ceec3c8d27 \
    JAVA_HOME="/usr/lib/jvm/java-11" \
    JAVA_VENDOR="openjdk" \
    JAVA_VERSION="11"

RUN microdnf --setopt=install_weak_deps=0 --setopt=tsflags=nodocs install -y apr-devel autoconf automake bc buildah bzip2-devel cmake diffutils file findutils gcc gcc-c++ git glibc-devel glibc-devel.i686 glibc-langpack-en glibc-static golang gzip hostname java-11-openjdk-devel libcurl-devel libgcc.i686 libstdc++-static libtool lsof make openssl-devel podman shadow-utils tar unzip wget which zlib-devel \
    && microdnf clean all \
    && rpm -q apr-devel autoconf automake bc buildah bzip2-devel cmake diffutils file findutils gcc gcc-c++ git glibc-devel glibc-devel.i686 glibc-langpack-en glibc-static golang gzip hostname java-11-openjdk-devel libcurl-devel libgcc.i686 libstdc++-static libtool lsof make openssl-devel podman shadow-utils tar unzip wget which zlib-devel

RUN microdnf --setopt=install_weak_deps=0 --setopt=tsflags=nodocs install -y emacs-filesystem vim-filesystem
# XXX: RHEL 8 does not have ninja-build, so grabbing from CentOS 8
RUN rpm --import https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Official \
    && rpm -q gpg-pubkey-8483c65d-5ccc5b19
RUN wget --no-verbose https://vault.centos.org/centos/8/PowerTools/x86_64/os/Packages/ninja-build-1.8.2-1.el8.x86_64.rpm \
    && rpm -K ninja-build-1.8.2-1.el8.x86_64.rpm \
    && rpm -ivh ninja-build-1.8.2-1.el8.x86_64.rpm \
    && rm ninja-build-1.8.2-1.el8.x86_64.rpm \
    && rpm -q ninja-build
RUN wget --no-verbose https://vault.centos.org/centos/8/AppStream/x86_64/os/Packages/git-lfs-2.11.0-2.el8.x86_64.rpm \
    && rpm -K git-lfs-2.11.0-2.el8.x86_64.rpm \
    && rpm -ivh git-lfs-2.11.0-2.el8.x86_64.rpm \
    && rm git-lfs-2.11.0-2.el8.x86_64.rpm \
    && rpm -q git-lfs

RUN mkdir -p /usr/share/maven/ref && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-3.8.7-bin.tar.gz   && echo "21c2be0a180a326353e8f6d12289f74bc7cd53080305f05358936f3a1b6dd4d91203f4cc799e81761cf5c53c5bbe9dcc13bdb27ec8f57ecf21b2f9ceec3c8d27  /tmp/apache-maven.tar.gz" | sha512sum -c -   && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1   && rm -f /tmp/apache-maven.tar.gz   && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

RUN set -o errexit -o nounset \
    && echo "Downloading Gradle Manipulator" \
    && wget --no-verbose --output-document=cli.jar "https://repo1.maven.org/maven2/org/jboss/gm/cli/3.11/cli-3.11.jar" \
    && wget --no-verbose --output-document=analyzer-init.gradle "https://repo1.maven.org/maven2/org/jboss/gm/analyzer/3.11/analyzer-3.11-init.gradle" \
    \
    && echo "Checking cli.jar download hash" \
    && echo "${CLI_JAR_DOWNLOAD_SHA256} *cli.jar" | sha256sum --check - \
    \
    && echo "Checking analyzer-init.gradle download hash" \
    && echo "${ANALYZER_INIT_DOWNLOAD_SHA256} *analyzer-init.gradle" | sha256sum --check - \
    \
    && echo "Installing Gradle Manipulator" \
    && mkdir "${GRADLE_MANIPULATOR_HOME}" \
    && mv cli.jar analyzer-init.gradle "${GRADLE_MANIPULATOR_HOME}/" \
    && mkdir "${GRADLE_MANIPULATOR_HOME}/bin" \
    && echo -e "#!/bin/sh\n\nset -eu\nARGS=\"-l \${GRADLE_HOME}\"\njava -jar ${GRADLE_MANIPULATOR_HOME}/cli.jar \"\${ARGS}\" --init-script=${GRADLE_MANIPULATOR_HOME}/analyzer-init.gradle \$@" > ${GRADLE_MANIPULATOR_HOME}/bin/gradle-manipulator \
    && chmod 755 ${GRADLE_MANIPULATOR_HOME}/bin/gradle-manipulator \
    && ln --symbolic "${GRADLE_MANIPULATOR_HOME}/bin/gradle-manipulator" /usr/bin/gradle-manipulator
    #we test out gradle manipulator as part of the gradle installation


RUN set -o errexit -o nounset \
    && mkdir /opt/gradle \
    && mkdir /opt/sbt \
    && mkdir /opt/ant \
         && echo "Downloading Ant 1.10.13" \
    && wget --no-verbose --output-document=ant.zip "https://dlcdn.apache.org/ant/binaries/apache-ant-1.10.13-bin.zip" \
    \
    && echo "Checking download hash 1.10.13" \
    && echo "800238184231f8002210fd0c75681bc20ce12f527c9b1dcb95fc8a33803bfce1 *ant.zip" | sha256sum --check - \
    \
    && echo "Installing Ant 1.10.13" \
    && unzip ant.zip \
    && rm ant.zip \
    && mv "apache-ant-1.10.13" "/opt/ant/1.10.13/" \
    && export ANT_HOME=/opt/ant/1.10.13 \
    && cd $ANT_HOME \
    && for i in antunit ivy logging junit xml networking regexp antlr bcel jdepend bsf debugging script javamail jspc; do $ANT_HOME/bin/ant -f fetch.xml -Ddest=system $i; done     && echo "Downloading SBT 1.8.0" \
    && wget --no-verbose --output-document=sbt.zip "https://github.com/sbt/sbt/releases/download/v1.8.0/sbt-1.8.0.zip" \
    \
    && echo "Checking download hash 1.8.0" \
    && echo "fb52ea0bc0761176f3e38923ae5df556fba372895efb98a587f706d1ae805897 *sbt.zip" | sha256sum --check - \
    \
    && echo "Installing SBT 1.8.0" \
    && unzip sbt.zip \
    && rm sbt.zip \
    && mv "sbt" "/opt/sbt/1.8.0/" \
    && export SBT_HOME=/opt/sbt/1.8.0     && echo "Downloading Gradle 8.0.2" \
    && wget --no-verbose --output-document=gradle.zip "https://services.gradle.org/distributions/gradle-8.0.2-bin.zip" \
    \
    && echo "Checking download hash 8.0.2" \
    && echo "ff7bf6a86f09b9b2c40bb8f48b25fc19cf2b2664fd1d220cd7ab833ec758d0d7 *gradle.zip" | sha256sum --check - \
    \
    && echo "Installing Gradle 8.0.2" \
    && unzip gradle.zip \
    && rm gradle.zip \
    && mv "gradle-8.0.2" "/opt/gradle/8.0.2/" \
    && export GRADLE_HOME=/opt/gradle/8.0.2; gradle-manipulator --version     && echo "Downloading Gradle 7.4.2" \
    && wget --no-verbose --output-document=gradle.zip "https://services.gradle.org/distributions/gradle-7.4.2-bin.zip" \
    \
    && echo "Checking download hash 7.4.2" \
    && echo "29e49b10984e585d8118b7d0bc452f944e386458df27371b49b4ac1dec4b7fda *gradle.zip" | sha256sum --check - \
    \
    && echo "Installing Gradle 7.4.2" \
    && unzip gradle.zip \
    && rm gradle.zip \
    && mv "gradle-7.4.2" "/opt/gradle/7.4.2/" \
    && export GRADLE_HOME=/opt/gradle/7.4.2; gradle-manipulator --version     && echo "Downloading Gradle 6.9.2" \
    && wget --no-verbose --output-document=gradle.zip "https://services.gradle.org/distributions/gradle-6.9.2-bin.zip" \
    \
    && echo "Checking download hash 6.9.2" \
    && echo "8b356fd8702d5ffa2e066ed0be45a023a779bba4dd1a68fd11bc2a6bdc981e8f *gradle.zip" | sha256sum --check - \
    \
    && echo "Installing Gradle 6.9.2" \
    && unzip gradle.zip \
    && rm gradle.zip \
    && mv "gradle-6.9.2" "/opt/gradle/6.9.2/" \
    && export GRADLE_HOME=/opt/gradle/6.9.2; gradle-manipulator --version     && echo "Downloading Gradle 5.6.4" \
    && wget --no-verbose --output-document=gradle.zip "https://services.gradle.org/distributions/gradle-5.6.4-bin.zip" \
    \
    && echo "Checking download hash 5.6.4" \
    && echo "1f3067073041bc44554d0efe5d402a33bc3d3c93cc39ab684f308586d732a80d *gradle.zip" | sha256sum --check - \
    \
    && echo "Installing Gradle 5.6.4" \
    && unzip gradle.zip \
    && rm gradle.zip \
    && mv "gradle-5.6.4" "/opt/gradle/5.6.4/" \
    && export GRADLE_HOME=/opt/gradle/5.6.4; gradle-manipulator --version     && echo "Downloading Gradle 4.10.3" \
    && wget --no-verbose --output-document=gradle.zip "https://services.gradle.org/distributions/gradle-4.10.3-bin.zip" \
    \
    && echo "Checking download hash 4.10.3" \
    && echo "8626cbf206b4e201ade7b87779090690447054bc93f052954c78480fa6ed186e *gradle.zip" | sha256sum --check - \
    \
    && echo "Installing Gradle 4.10.3" \
    && unzip gradle.zip \
    && rm gradle.zip \
    && mv "gradle-4.10.3" "/opt/gradle/4.10.3/" \
    && export GRADLE_HOME=/opt/gradle/4.10.3; gradle-manipulator --version

LABEL \
    io.cekit.version="4.1.0" \
    io.k8s.description="HACBS builder image for building Java components." \
    io.k8s.display-name="HACBS BuilderImage" \
    io.openshift.tags="builder,java,hacbs" \
    maintainer="Stuart Douglas <sdouglas@redhat.com>"

# Clear package manager metadata
RUN microdnf clean all && [ ! -d /var/cache/yum ] || rm -rf /var/cache/yum && mkdir /project && chown 185:185 /project
# Define the user
USER 185
# Define the working directory
WORKDIR /project
