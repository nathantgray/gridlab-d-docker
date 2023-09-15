FROM debian:bookworm AS base

ENV DEBIAN_FRONTEND="noninteractive" \
    TZ=UTC+0 \
    HELICS_INSTALL_PATH=/helics \
    GLD_INSTALL_PATH=/gridlabd \
    OPTIMISER_LEVEL=0


FROM base AS builder

# GridLAB-D required environment variables:
ENV GLPATH=${GLD_INSTALL_PATH}/lib/gridlabd:${GLD_INSTALL_PATH}/share/gridlabd \
    LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${HELICS_INSTALL_PATH}/lib \
    PATH=${PATH}:${GLD_INSTALL_PATH}:${HELICS_INSTALL_PATH}

# Build dependencies.
RUN apt-get -y update \
    && apt-get -y install --no-install-recommends \
      autoconf \
      automake \
      build-essential \
      ca-certificates \
      cmake \
      default-libmysqlclient-dev \
      g++ \
      gcc \
      git \
      libboost-dev \
      libboost-chrono-dev \
      libboost-date-time-dev \
      libboost-filesystem-dev \
      libboost-program-options-dev \
      libboost-test-dev \
      libboost-timer-dev \
      libtool \
      libxerces-c3.2 \
      libxerces-c-dev \
      libzmq5-dev \
      make \
      swig \
      python3-dev \
      python3-pip

WORKDIR /build

# Build Helics, dynamically linked by GridLab-D bellow.
RUN git \
    -c advice.detachedHead=false \
    clone https://github.com/GMLC-TDC/HELICS.git \
    --branch v3.4.0 \
    --depth 1 \
    --single-branch \
    ./HELICS

WORKDIR /build/HELICS/build

RUN cmake \
      -D HELICS_BUILD_CXX_SHARED_LIB=ON \
      -D CMAKE_INSTALL_PREFIX=${HELICS_INSTALL_PATH} \
      -S ../

#RUN cmake --build . -j8 --target install
RUN make -j $(nproc)
RUN make -j $(nproc) install

WORKDIR /build


# Build GridLab-D.
RUN git \
    -c advice.detachedHead=false \
    clone https://github.com/gridlab-d/gridlab-d.git \
    --branch v5.1 \
    --depth 1 \
    --single-branch \
    ./gridlab-d
WORKDIR /build/gridlab-d
RUN mkdir cmake-build
WORKDIR /build/gridlab-d/cmake-build
RUN git submodule update --init
RUN ls
RUN cmake -S /build/gridlab-d -DCMAKE_INSTALL_PREFIX=${GLD_INSTALL_PATH} -DGLD_USE_HELICS=ON -DHELICS_DIR=${HELICS_INSTALL_PATH} -DCMAKE_VERBOSE_MAKEFILE=ON ..

RUN cmake --build . -j8 --target install
#RUN autoreconf -if

#RUN ./configure \
#        --prefix=${GLD_INSTALL_PATH} \
#        --with-mysql=/usr/lib/x86_64-linux-gnu/ \
#        --with-helics=${HELICS_INSTALL_PATH} \
#        --enable-silent-rules \
#        "CFLAGS=-g -O${OPTIMISER_LEVEL} -w" "CXXFLAGS=-g -O${OPTIMISER_LEVEL} -w -std=c++14" "LDFLAGS=-g -O${OPTIMISER_LEVEL} -w"

#RUN make MYSQL_CPPFLAGS='-I/usr/include/mysql/' --debug=v
#RUN make install


# Create our runner image without all the build baggage.
FROM base AS runner

# Copy over the built paths
COPY --from=builder ${HELICS_INSTALL_PATH} /usr/local/
COPY --from=builder ${GLD_INSTALL_PATH} /usr/local/

# GridLAB-D required environment variable
ENV GLPATH=/usr/local/lib/gridlabd:/usr/local/share/gridlabd
ENV PYTHONPATH /usr/local/python

# Run-time dependencies.
RUN apt-get -y update \
    && apt-get -y install --no-install-recommends \
        default-libmysqlclient-dev \
        libboost-dev \
        libboost-filesystem1.74.0 \
        libboost-program-options1.74.0 \
        libboost-test1.74.0 \
        libxerces-c3.2 \
        libzmq5 \
        python3-dev  \
        python3-pip \
        python3-venv \
        pipx \
    && rm -rf /var/lib/apt/lists/*


RUN python3 -m venv /venv
ENV PATH="/venv/bin:$PATH"
RUN pip install --upgrade pip
RUN pip install helics

#RUN pipx install virtualenv
#RUN pipx ensurepath
#RUN virtualenv venv
#RUN python3 -m pip install --user virtualenv
#RUN python3 -m virtualenv venv
# Python must be installed after the PYTHONPATH is set above for it to
# recognize and import libhelicsSharedLib.so.
#RUN pip install helics
ENTRYPOINT ["bash"]
