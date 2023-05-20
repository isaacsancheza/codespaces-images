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
    && apt-get install -y --no-install-recommends git zsh \
    #
	# add non-root user
	&& groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/zsh --uid $USER_UID --gid $USER_GID -m $USERNAME \
	#
    # clean-up
    && apt-get autoremove -y  \
    && apt-get clean -y  \
    && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# install todo.sh
COPY --from=BASE /usr/local/bin/todo.sh /usr/local/bin/todo.sh

COPY --from=BASE /usr/share/bash-completion/completions/todo /usr/share/bash-completion/completions/todo

# copy oh-my-zsh installer
COPY --from=BASE $WORKDIR/omz-installer.sh $WORKDIR/omz-installer.sh

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