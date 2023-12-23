FROM debian:bullseye-slim
MAINTAINER Abdelmajid Elhamdaoui <elhamdaouiabdelmajid@gmail.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
# Install system dependencies

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gcc \
    git \
    gnupg \
    lsb-release \
    nodejs \
    npm \
    python3-dev \
    python3-pip \
    python3-venv \
    libpq-dev \
    libldap2-dev \
    libsasl2-dev \
    libssl-dev \
    build-essential \
    libxml2-dev \
    libxslt1-dev \
    libjpeg-dev \
    zlib1g-dev \
    libfreetype6-dev \
    liblcms2-dev \
    libwebp-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libtiff5-dev \
    libpng-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


# Upgrade pip
RUN pip3 install --upgrade pip wheel
RUN pip3 install cython
RUN pip3 install gevent
RUN pip3 install greenlet
RUN pip3 install lxml

#INSTALL WKHTMLTOPDF
ARG TARGETARCH

RUN apt-get update && apt-get install -y \
    fontconfig \
    libx11-6 \
    libxcb1 \
    libxext6 \
    libxrender1 \
    xfonts-75dpi \
    xfonts-base \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN if [ -z "${TARGETARCH}" ]; then \
        TARGETARCH="$(dpkg --print-architecture)"; \
    fi; \
    WKHTMLTOPDF_ARCH=${TARGETARCH} && \
    case ${TARGETARCH} in \
    "amd64") WKHTMLTOPDF_ARCH=amd64 && WKHTMLTOPDF_SHA=9df8dd7b1e99782f1cfa19aca665969bbd9cc159  ;; \
    "arm64")  WKHTMLTOPDF_SHA=58c84db46b11ba0e14abb77a32324b1c257f1f22  ;; \
    "ppc64le" | "ppc64el") WKHTMLTOPDF_ARCH=ppc64el && WKHTMLTOPDF_SHA=7ed8f6dcedf5345a3dd4eeb58dc89704d862f9cd  ;; \
    esac \
    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.bullseye_${WKHTMLTOPDF_ARCH}.deb \
    && echo ${WKHTMLTOPDF_SHA} wkhtmltox.deb | sha1sum -c - \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ bullseye-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

# Install Odoo
RUN adduser --system --quiet --shell=/bin/bash --home=/opt/odoo --gecos 'ODOO' --group odoo
RUN adduser odoo sudo

RUN apt-get update -y
RUN apt-get install git -y 

RUN mkdir -p /opt/odoo/odoo-server \
    && chown -R odoo /opt/odoo/odoo-server

ENV ODOO_VERSION 12.0
ARG ODOO_RELEASE=20231222
ARG ODOO_SHA=475688179c3857aa76c46f86486d848812ba046b
RUN git clone -b ${ODOO_VERSION} https://github.com/odoo/odoo --depth 1 /opt/odoo/odoo-server



#RUN pip3 install -r /opt/odoo/odoo-server/requirements.txt
RUN grep -v 'gevent' /opt/odoo/odoo-server/requirements.txt > /opt/odoo/odoo-server/filtered-requirements1.txt
RUN grep -v 'suds-jurko' /opt/odoo/odoo-server/filtered-requirements1.txt > /opt/odoo/odoo-server/filtered-requirements.txt
RUN grep -v 'greenlet' /opt/odoo/odoo-server/filtered-requirements.txt > /opt/odoo/odoo-server/filtered-requirements1.txt
RUN grep -v 'lxml' /opt/odoo/odoo-server/filtered-requirements1.txt > /opt/odoo/odoo-server/filtered-requirements.txt
RUN grep -v 'reportlab' /opt/odoo/odoo-server/filtered-requirements.txt > /opt/odoo/odoo-server/filtered-requirements1.txt

RUN pip3 install 'setuptools<58.0.0'
RUN pip3 install 'suds-jurko==0.6'
RUN pip3 install --upgrade setuptools
RUN pip3 install 'reportlab>=3.5.0'
RUN pip3 install -r /opt/odoo/odoo-server/filtered-requirements1.txt
RUN rm /opt/odoo/odoo-server/filtered-requirements1.txt
RUN rm /opt/odoo/odoo-server/filtered-requirements.txt


# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo: /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo: /mnt/extra-addons

RUN mkdir -p /var/lib/odoo \
    && chown -R odoo: /var/lib/odoo

RUN chown -R odoo /opt/odoo
RUN chmod 777 -R /opt/odoo
RUN chmod 777 -R /var/lib/odoo
RUN chmod 777 -R /mnt/extra-addons


VOLUME ["/var/lib/odoo", "/opt/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/opt/odoo/odoo-server/odoo-bin "]
