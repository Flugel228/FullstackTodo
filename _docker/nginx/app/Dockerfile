FROM nginx:1.21.6 AS builder

RUN apt-get update && \
    apt-get install -y \
        build-essential \
        cmake \
        git \
        wget \
        unzip \
        zlib1g-dev \
        libpcre3 \
        libpcre3-dev \
        uuid-dev && \
    rm -rf /var/lib/apt/lists/*

RUN cd /tmp && \
    git clone https://github.com/google/brotli.git && \
    cd brotli && \
    mkdir out && \
    cd out && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local .. && \
    make -j$(nproc) && \
    make install

RUN cd /tmp && \
    wget http://nginx.org/download/nginx-1.21.6.tar.gz && \
    tar -xzf nginx-1.21.6.tar.gz && \
    git clone https://github.com/google/ngx_brotli.git && \
    cd ngx_brotli && \
    git submodule update --init && \
    cd /tmp/nginx-1.21.6 && \
    ./configure \
        --with-compat \
        --add-dynamic-module=../ngx_brotli && \
    make modules && \
    ls -l objs/*.so

FROM nginx:1.21.6

COPY --from=builder /tmp/nginx-1.21.6/objs/ngx_http_brotli_filter_module.so /etc/nginx/modules/
COPY --from=builder /tmp/nginx-1.21.6/objs/ngx_http_brotli_static_module.so /etc/nginx/modules/

RUN sed -i '/^events {/i load_module /etc/nginx/modules/ngx_http_brotli_filter_module.so;' /etc/nginx/nginx.conf && \
    sed -i '/^events {/i load_module /etc/nginx/modules/ngx_http_brotli_static_module.so;' /etc/nginx/nginx.conf

CMD ["nginx", "-g", "daemon off;"]
