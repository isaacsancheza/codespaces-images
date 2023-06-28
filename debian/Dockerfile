FROM debian:bullseye AS BASE

ARG WORKDIR=/codespace

RUN apt-get update  \
    && export DEBIAN_FRONTEND=noninteractive \
    #
    # update certificates
    && apt-get install -y --reinstall ca-certificates \
    # 
    # install zsh
    && apt-get install -y --no-install-recommends git curl build-essential \
    #
    # clean-up/
    && apt-get autoremove -y  \
    && apt-get clean -y  \
    && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# install todo.sh
RUN git clone https://github.com/todotxt/todo.txt-cli todo \
    #
    && export INSTALL_DIR=/usr/local/bin/ \
    #
    && export CONFIG_DIR=$HOME/.todo/config \
    #
    # why this should be installed there
    # https://unix.stackexchange.com/questions/416185/where-to-install-bash-completion-scripts-for-out-of-tree-packages
    && export BASH_COMPLETION=/usr/share/bash-completion/completions/ \
    #
    && cd todo \
    && make \
    && make install

# cd /codespace
WORKDIR $WORKDIR

# download oh-my-zsh
RUN curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -o omz-installer.sh


FROM debian:bullseye AS IMAGE

ARG USERNAME=codespace
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG WORKDIR=/codespace

RUN apt-get update  \
    && export DEBIAN_FRONTEND=noninteractive \
    #
    # update certificates
    && apt-get install -y --reinstall ca-certificates \
    # 
    # install zsh
    && apt-get install -y --no-install-recommends acl git zip zsh vim curl less wget groff unzip \
    #
	# add non-root user
	&& groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/zsh --uid $USER_UID --gid $USER_GID -m $USERNAME \
	#
    # clean-up
    && apt-get autoremove -y  \
    && apt-get clean -y  \
    && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# install github-cli
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && apt update \
    && apt install -y gh

# install todo.sh
COPY --from=BASE /usr/local/bin/todo.sh /usr/local/bin/todo.sh
COPY --from=BASE /usr/share/bash-completion/completions/todo /usr/share/bash-completion/completions/todo

# copy oh-my-zsh installer
COPY --from=BASE $WORKDIR/omz-installer.sh $WORKDIR/omz-installer.sh

# download and install aws-cli
RUN curl -sL https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o aws-cli.zip \
    && unzip aws-cli.zip \
    && ./aws/install \
    && rm -rf aws-cli.zip aws

# download and install sam-cli
RUN curl -sL https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip -o aws-sam-cli.zip \
    && unzip aws-sam-cli.zip -d sam-installation \
    && ./sam-installation/install \
    && rm -rf aws-sam-cli.zip sam-installation

# cd /workspace
WORKDIR $WORKDIR

# chown oh-my-zsh installer
RUN chown -R $USERNAME:$USERNAME $WORKDIR

# install oh-my-zsh as non-root user
USER $USERNAME

# install oh-my-zsh
RUN sh omz-installer.sh \
    && rm omz-installer.sh

# install powerlevel10k theme
RUN git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# go back to root user
USER root

# todo.sh permissions
RUN chmod 644 /usr/share/bash-completion/completions/todo

# chown /codespace
RUN chown -R $USERNAME:$USERNAME $WORKDIR

# set default file permissions for workspace directory
# https://www.baeldung.com/linux/new-files-dirs-default-permission
# +X for directories only and not files
# https://unix.stackexchange.com/questions/1314/how-to-set-default-file-permissions-for-all-folders-files-in-a-directory
RUN setfacl -PRdm u::rwX,g::rwX,o::r $WORKDIR
