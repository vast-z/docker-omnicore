FROM ubuntu:latest as builder
LABEL maintainer="vast <vast.z@outlook.com>"

ENV BITCOIN_ROOT=/omnicore
ENV BDB_PREFIX="${BITCOIN_ROOT}/db4" BITCOIN_REPO="${BITCOIN_ROOT}/repo" PATH="${BITCOIN_ROOT}/bin:$PATH" BITCOIN_DATA="${BITCOIN_ROOT}/data"

RUN mkdir -p $BITCOIN_ROOT && mkdir -p $BDB_PREFIX

WORKDIR ${BITCOIN_ROOT}

RUN apt update && apt-get install -y build-essential libtool autotools-dev automake pkg-config bsdmainutils python3 libboost-all-dev
RUN apt-get install -y openssl libssl-dev
RUN apt-get install -y libevent-dev

# clone omnicore
RUN git clone -b v0.8.1 https://github.com/OmniLayer/omnicore.git $BITCOIN_REPO
# COPY omnicore $BITCOIN_REPO

# db
RUN  wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz' && \
    echo '12edc0df75bf9abd7f82f821795bcee50f42cb2e5f76a6a281b85732798364ef  db-4.8.30.NC.tar.gz' | sha256sum -c
# COPY ./db-4.8.30.NC.tar.gz ./db-4.8.30.NC.tar.gz

RUN tar -xzf db-4.8.30.NC.tar.gz
RUN cd db-4.8.30.NC/build_unix/ && \
    ../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX && \
    make -j4 && \
    make install
RUN cd $BITCOIN_REPO && \
    ./autogen.sh && \
    ./configure \
        LDFLAGS="-L${BDB_PREFIX}/lib/" \
        CPPFLAGS="-I${BDB_PREFIX}/include/" \
        --disable-tests \
        --disable-bench \
        --disable-ccache \
        --with-gui=no \
        --with-utils \
        --with-libs \
        --with-daemon \
        --without-gui \
        --prefix=$BITCOIN_ROOT && \
    make -j4 && \
    make install && \
    rm -rf $BITCOIN_ROOT/db-4.8.30.NC* && \
    rm -rf $BDB_PREFIX/docs && \
    rm -rf $BITCOIN_REPO && \
    strip $BITCOIN_ROOT/bin/omnicore-cli && \
    strip $BITCOIN_ROOT/bin/bitcoin-tx && \
    strip $BITCOIN_ROOT/bin/omnicored && \
    strip $BITCOIN_ROOT/lib/libbitcoinconsensus.a && \
    strip $BITCOIN_ROOT/lib/libbitcoinconsensus.so.0.0.0

WORKDIR ${BITCOIN_DATA}
VOLUME ["${BITCOIN_DATA}"]

COPY docker-entrypoint.sh /entrypoint.sh

RUN chmod u+x /entrypoint.sh

EXPOSE 8332 8333 18332 18333 18444

ENTRYPOINT ["/entrypoint.sh"]

CMD ["omnicored"]
