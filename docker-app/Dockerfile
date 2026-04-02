# ============================================
#  INTENTIONALLY VULNERABLE - DO NOT USE!
# ============================================

# VULN: Using old Ubuntu with known CVEs + unpinned tag
FROM ubuntu:20.04

# VULN: Running as root (no USER directive)

# VULN: Using old Node.js with known CVEs
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    openssl \
    && curl -fsSL https://deb.nodesource.com/setup_14.x | bash - \
    && apt-get install -y nodejs

# VULN: Copying everything including secrets
COPY . /app

WORKDIR /app

# VULN: Installing without lock file
RUN npm install

# VULN: Exposing debug port
EXPOSE 3000 9229

CMD ["node", "index.js"]
