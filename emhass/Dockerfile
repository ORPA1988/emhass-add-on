## EMHASS Add-On - Built from ORPA1988 fork with publish-data fix
ARG BUILD_FROM
FROM ${BUILD_FROM}

# Install git + build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    git curl ca-certificates gnupg \
    gcc g++ patchelf cmake ninja-build \
    libgfortran5 libopenblas0-pthread libopenblas-dev \
    libatlas3-base libatlas-base-dev \
    libsz2 libaec0 libhdf5-hl-100 libhdf5-103-1 libhdf5-dev libhdf5-serial-dev \
    tini \
    && rm -rf /var/lib/apt/lists/* \
    && curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR="/usr/local/bin" sh \
    && uv python install \
    && ln -s /usr/include/hdf5/serial /usr/include/hdf5/include \
    && mkdir -p /data/ /share/emhass/

WORKDIR /app

# Clone the fixed EMHASS fork
RUN git clone --depth 1 https://github.com/ORPA1988/emhass.git /app

# Setup venv and install
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
RUN uv venv
ENV PATH="/app/.venv/bin:$PATH"
RUN uv pip install --verbose .

# Remove build-only packages
RUN apt-get remove --purge -y --auto-remove gcc g++ patchelf cmake ninja-build git \
    && rm -rf /var/lib/apt/lists/*

# Copy default data files
RUN cp /app/data/opt_res_latest.csv /data/ 2>/dev/null || true \
    && cp /app/data/long_train_data.pkl /data/ 2>/dev/null || true

ENV WORKER_CLASS=uvicorn.workers.UvicornWorker PORT=5000 IP=0.0.0.0

ENTRYPOINT ["/usr/bin/tini", "--", "/bin/bash", "-c", "set -e && \
if [ ! -f /data/long_train_data.pkl ] && [ -f /app/data/long_train_data.pkl ]; then \
    cp /app/data/long_train_data.pkl /data/; \
fi && \
if [ ! -f /data/opt_res_latest.csv ] && [ -f /app/data/opt_res_latest.csv ]; then \
    cp /app/data/opt_res_latest.csv /data/; \
fi && \
exec uv run --frozen gunicorn emhass.web_server:app -c gunicorn.conf.py -k \"$WORKER_CLASS\""]
