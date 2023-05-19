FROM debian:bullseye AS BASE

ARG WORKDIR=/workspace

RUN apt-get update  \
    && export DEBIAN_FRONTEND=noninteractive \
    #
    # Update certificates
    && apt-get install -y --reinstall ca-certificates \
    # 
    # Install zsh
    && apt-get install -y --no-install-recommends git curl build-essential \
    #
    # clean-up/
    && apt-get autoremove -y  \
    && apt-get clean -y  \
    && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# cd /workspace
WORKDIR $WORKDIR

# Install todo.sh
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

# Download oh-my-zsh
RUN curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -o omz-installer.sh


FROM debian:bullseye AS IMAGE

ARG WORKDIR=/workspace

RUN apt-get update  \
    && export DEBIAN_FRONTEND=noninteractive \
    #
    # Update certificates
    && apt-get install -y --reinstall ca-certificates \
    # 
    # Install zsh
    && apt-get install -y --no-install-recommends git zsh \
    #
    # clean-up
    && apt-get autoremove -y  \
    && apt-get clean -y  \
    && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# Install todo.sh
COPY --from=BASE /usr/local/bin/todo.sh /usr/local/bin/todo.sh

# Why copy there
COPY --from=BASE /usr/share/bash-completion/completions/todo /usr/share/bash-completion/completions/todo

# Copy oh-my-zsh installer
COPY --from=BASE $WORKDIR/omz-installer.sh $WORKDIR/omz-installer.sh

# cd /workspace
WORKDIR $WORKDIR

# Install oh-my-zsh
RUN sh omz-installer.sh \
    && rm omz-installer.sh

# todo.sh permissions
RUN chmod 644 /usr/share/bash-completion/completions/todo
