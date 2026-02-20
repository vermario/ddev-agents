#ddev-generated
# Install and configure SSH server for remote command execution from agents container

# Install SSH server
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        openssh-server \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy custom sshd configuration
COPY sshd_config /etc/ssh/sshd_config

# Generate host keys and register sshd as a supervised service
# Note: supervisord runs as the DDEV user, so sudo is needed for sshd (port 22, host key access)
# /run/sshd is on tmpfs and must be created at runtime
RUN ssh-keygen -A && \
    printf '[program:sshd]\ncommand=sudo bash -c "mkdir -p /run/sshd && exec /usr/sbin/sshd -D"\nautorestart=true\nstartretries=3\n' \
        > /etc/supervisor/conf.d/sshd.conf
