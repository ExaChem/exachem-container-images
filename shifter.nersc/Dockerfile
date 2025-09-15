ARG         UBUNTU_TAG
ARG         GITHUB_REPOSITORY_OWNER
ARG         GITHUB_REPOSITORY
#

FROM $UBUNTU_TAG

LABEL maintainer="Edoardo Apra <edoardo.apra@pnnl.gov>"

LABEL org.opencontainers.image.description "Exachem CC"$CONTAINER"image built with Mpich 4.0"

#LABEL org.opencontainers.image.source "https://github.com/Exachem/exachem-container-images"

LABEL org.opencontainers.image.licenses "Apache-2.0"


#caching from host cache dir
COPY Dockerfile cache* /tmp/
COPY myaug.patch /tmp/

ARG DEBIAN_FRONTEND=noninteractive
ARG USE_OPENMP_VALUE
ARG         FC
ARG         TAMM_BRANCH
ARG         USE_SYSV
ARG         USE_CPU
ARG         CONTAINER
ENV         FC=${FC}
ENV         TAMM_BRANCH=${TAMM_BRANCH}
ENV         GITHUB_REPOSITORY_OWNER=${GITHUB_REPOSITORY_OWNER}
ENV         USE_SYSV=$USE_SYSV
ENV         USE_CPU=$USE_CPU
ENV         CONTAINER=${CONTAINER}
ARG mpich=4.0.2
ARG mpich_prefix=mpich-$mpich
ARG	    CUDA_VERSION_MAJOR=12 \
	    CUDA_VERSION_MINOR=4
ARG	    PATH=/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/cuda-"$CUDA_VERSION_MAJOR"."$CUDA_VERSION_MINOR"/bin:/opt/install/exachem/bin:/usr/local/bin \
	    LD_LIBRARY_PATH=/usr/local/cuda-"$CUDA_VERSION_MAJOR"."$CUDA_VERSION_MINOR"/lib64:/usr/local/lib  
WORKDIR     /opt
#ENV CRAYPE_LINK_TYPE=dynamic
ENV	    CUDA_VERSION_MAJOR=12 
ENV	    CUDA_VERSION_MINOR=4
ENV         GCC_VERSION=12
ENV         PY_VERSION=3.11
ENV	    PATH=/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/cuda-"$CUDA_VERSION_MAJOR"."$CUDA_VERSION_MINOR"/bin:/opt/nwchem/bin:/usr/local/bin \
	    LD_LIBRARY_PATH=/usr/local/cuda-"$CUDA_VERSION_MAJOR"."$CUDA_VERSION_MINOR"/lib64:/usr/local/lib  
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN           env && if [[ "${CONTAINER}" == shifter ]]; then groupadd -r exachem -g 1994 ;\
              useradd  -u 1994 -r -g exachem -c 'Exachem user' -m -d /opt/install/exachem -s /bin/bash exachem ; fi && \
            if [[ "${CONTAINER}" == shifter ]]; then export MYUSER=exachem ; else export MYUSER=root ;fi && \
    mkdir -p /opt \
    && ls -lart /tmp \
&& { echo "line=\$(grep -hni ecp \$1|head -1|cut -d : -f 1) " ; \
echo " linem1=\$((line-2)) " ; \
echo " rm -f az_ecp " ; \
echo " head -n \$linem1 \$1 > az_ecp " ; \
echo " mv az_ecp \$1 " ; }  > /opt/ecp_strip.sh && \
chmod +x /opt/ecp_strip.sh && \
cat /opt/ecp_strip.sh && \
  { echo "line=\$(grep -hni ecp \$1|head -1|cut -d : -f 1) " ; \
echo " length=\$(wc -l \$1 | cut -d ' ' -f 1) " ; \
echo " linem1=\$((length-line+1)) " ; \
echo " rm -f az_ecp " ; \
echo " tail -n \$linem1 \$1 > az_ecp " ; \
echo " mv az_ecp \$1" ; } > /opt/ao_strip.sh && \
chmod +x  /opt/ao_strip.sh && \
cat /opt/ao_strip.sh && \
         apt-get -q=2 update \
            && apt-get -q=2 -y upgrade \
            && apt-get install -q=2 -y  gcc-"$GCC_VERSION"  g++-"$GCC_VERSION" gfortran-"$GCC_VERSION"   make ssh patch rsync wget curl git file python"$PY_VERSION" libpython"$PY_VERSION"-dev python"$PY_VERSION"-dev bzip2 tar gnupg2 cmake \
            && apt-get install -q=2 -y  autoconf automake libtool git  gdb \
	    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-"$GCC_VERSION" 100 \
	    && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-"$GCC_VERSION" 100 \
	    && update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-"$GCC_VERSION" 100 \
	    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 100 \
	    && update-alternatives --install /usr/bin/python3-config python3-config /usr/bin/python3.11-config 100 \
            && export FETCHED_LIBINT=0 \
	    && export FETCHED_MPICH=0 \
	    && if [[ -z "${FC}" ]]; then export FC=gfortran ; fi \
	    && echo FCFCFC is $FC >& nvcc.out && cat nvcc.out && rm nvcc.out \
	    && if [[ -z "${CC}" ]]; then export CC=gcc; fi \
	    && echo CCCCCC is $CC >& nvcc.out && cat nvcc.out && rm nvcc.out \ 
	    && if [[ -z "${CXX}" ]]; then export CXX=g++; fi \
	    && echo CXXCXX is $CXX >& nvcc.out && cat nvcc.out && rm nvcc.out \
	    && echo gfortran -v `gfortran -v` >& nvcc.out && cat nvcc.out && rm nvcc.out\
            && VERSION=$(gfortran -dumpversion  | cut -f1 -d. ) \
	    && echo "VERSION is " $VERSION   >& nvcc.out && cat nvcc.out && rm nvcc.out \
	    && echo "expr is " $(which expr)  >& nvcc.out && cat nvcc.out && rm nvcc.out \
            &&  VERSIONGT9=$(/usr/bin/expr "$VERSION"  \> 9)  || true \
	    && echo "VERSIONGT9 is " $VERSIONGT9   >& nvcc.out && cat nvcc.out && rm nvcc.out \
	    && if [[ "$FC" == gfortran ]] && [[ "$VERSIONGT9" == 1 ]]; then export FFLAGS_IN="-fallow-argument-mismatch"; else  FFLAGS_IN=" "; fi \
	    && echo "VERSIONGT9 is " $VERSIONGT9 \
	    && echo "FFLAGS_IN is " $FFLAGS_IN \
            && echo "FFLAGS_IN is " $FFLAGS_IN 	>& nvcc.out && cat nvcc.out && rm nvcc.out \
	    && echo GITHUB_REPOSITORY_OWNER is $GITHUB_REPOSITORY_OWNER \
            && if [[ $USE_CPU == N ]]; then echo "cuda install" \
	    ; wget --progress=dot:giga https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-archive-keyring.gpg \
            ; cp cuda-archive-keyring.gpg /usr/share/keyrings/cuda-archive-keyring.gpg \
            ; echo "deb [signed-by=/usr/share/keyrings/cuda-archive-keyring.gpg] https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/ /" |  tee /etc/apt/sources.list.d/cuda-ubuntu2204-x86_64.list \
            ;  wget --progress=dot:giga https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin \
            ; mv cuda-*.pin /etc/apt/preferences.d/cuda-repository-pin-600 \
	    ; apt-get -q=2 update \
            ; apt-get -q=2 -y install cuda-nvcc-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" cuda-cudart-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libcublas-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" cuda-nvml-dev-${CUDA_VERSION_MAJOR}-${CUDA_VERSION_MINOR} cuda-libraries-${CUDA_VERSION_MAJOR}-${CUDA_VERSION_MINOR} cuda-libraries-dev-${CUDA_VERSION_MAJOR}-${CUDA_VERSION_MINOR}  \
	    ; rm -f nvcc.out ; nvcc -V >& nvcc.out ; cat nvcc.out ; rm nvcc.out ; fi \
	    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-"$GCC_VERSION" 1000 \
	    && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-"$GCC_VERSION" 1000 \
	    && update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-"$GCC_VERSION" 1000 \
	    && env|egrep PATH \
            && arch=$(uname -m) \
	    && echo arch is "$arch" \
	    && arch_dpkg=$(dpkg --print-architecture)  \
	    && echo 'FC -v ' $("$FC" -v) \
            && apt-get clean \
	    && echo 'checking cache #2 ' \
            && if [[ -f /tmp/mpich.tar.bz2 ]]; then cd /usr/local ; tar xjf /tmp/mpich.tar.bz2; rm -rf /tmp/mpich.tar.bz2 ; export FETCHED_MPICH=1; \
    else  echo 'downloading mpich source' ; \
    wget --progress=dot:giga https://www.mpich.org/static/downloads/$mpich/$mpich_prefix.tar.gz -O - | tar xzf - ; \
    cd $mpich_prefix || { echo "Failure"; exit 1; } ;\
    FFLAGS=$FFLAGS_IN FCFLAGS=$FFLAGS_IN ./configure --silent --enable-silent-rules ; \
    echo 'redirecting output to make.log'                                   ; \
    make -j 4 FFLAGS=$FFLAGS_IN  >& make.log                 ; \
    echo executing "make -j 4 FFLAGS=$FFLAGS_IN  "                 ; \
    echo 'mpich compiled' ; \
    tail -300 make.log         ; \
    echo 'redirecting output to install.log'                                   ; \
    make install    >& install.log                                        ; \
    tail install.log ; \
    cd ..                                                                   ; \
    rm -rf $mpich_prefix ; echo 'mpich step done'; fi \
            && echo 'after mpich compilation ' \
            && rm -f mpicc.show && /usr/local/bin/mpicc -show >& mpicc.show && echo "mpicc show" `cat mpicc.show` && rm -f mpicc.show \
	    && echo TAMM_BRANCH is ${TAMM_BRANCH} \
	    && echo 'cloning tamm source *** ' || true \
	    && mkdir -p /opt/install/exachem/ /opt/cache\
            && if [[ -f /tmp/libint.tar.bz2 ]]; then cd /opt/install/exachem; tar xjvf /tmp/libint.tar.bz2; rm -rf /tmp/libint.tar.bz2 ; export FETCHED_LIBINT=1; fi \
	    && cd / && export CMAKE_VER=3.29.5 \
	    && curl -LJO https://github.com/Kitware/CMake/releases/download/v"$CMAKE_VER"/cmake-"$CMAKE_VER"-linux-x86_64.tar.gz && tar xzf cmake-"$CMAKE_VER"-linux-x86_64.tar.gz \
	    && export PATH=/cmake-"$CMAKE_VER"-linux-x86_64/bin:$PATH \
            && cd /opt || { echo "Failure"; exit 1; } \
            && git clone -b ${TAMM_BRANCH} --depth 100 https://github.com/NWChemEx/TAMM.git   \
            && git clone -b ${TAMM_BRANCH} --depth 20  https://github.com/ExaChem/exachem.git exachem  \
            && 	     cd TAMM   \
            && 	     mkdir build && cd build  \
	    && echo " USE_SYSV is " $USE_SYSV \
	    && echo " USE_CPU is " $USE_CPU \
            && CMAKE_OPTIONS="-DCMAKE_INSTALL_PREFIX=/opt/install/exachem -DMODULES=CC  -DBLIS_CONFIG=zen3 -DMARCH_FLAGS=-march=znver3 " \
	    && CMAKE_OPTIONS="$CMAKE_OPTIONS -DCMAKE_BUILD_TYPE=MinSizeRel " \
            && if [[ $USE_SYSV == Y ]]; then echo "sysv enabled"; CMAKE_OPTIONS+="-DGA_ENABLE_SYSV=ON "; fi \
            && if [[ $USE_CPU == N ]]; then echo "gpu code"; CMAKE_OPTIONS+="-DTAMM_ENABLE_CUDA=ON -DGPU_ARCH=80" ; else echo "cpu code" ;fi \
            && export CMAKE_OPTIONS \
	    && echo CMAKE_OPTIONS $CMAKE_OPTIONS \
	    && CC=gcc CXX=g++ FC=gfortran cmake  $CMAKE_OPTIONS .. \
            && make -j3 || { echo 'tamm Compilation Failure'; exit 1; }  \
	    && echo exit code for compilation $? \
            && make install \
	    && echo exit code for make install $? \
            && cd /opt/exachem \
	    && patch -p1 -s -N < /tmp/myaug.patch \
	    && mkdir build && cd build \
	    && CC=gcc CXX=g++ FC=gfortran cmake  $CMAKE_OPTIONS .. \
            && make -j3 || { echo 'exachem Compilation Failure'; exit 1; }  \
            && make install \
            && bash -c "if [[ -f /opt/install/exachem/bin/ExaChem ]]; then echo 'Exachem binary created' ; else  echo 'compilation error1'; exit 1; fi"  \
	    && echo exit code for make install $? \
	    && echo git describe --always `git describe --always`  >> /opt/install/info.txt \
	    && echo git rev-parse HEAD git `git rev-parse HEAD` >> /opt/install/info.txt \
	    && cd /opt/install/exachem/share/libint/*/basis/ \
	    && rm -f aug* cc-p*v?z* || true \
	    && wget   --progress=dot:giga -O cc-pvdz-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pvdz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pvtz-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pvtz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pvqz-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pv5z-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pv5z-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pvdz.g94 "https://www.basissetexchange.org/api/basis/cc-pvdz/format/gaussian94/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pvtz.g94 "https://www.basissetexchange.org/api/basis/cc-pvtz/format/gaussian94/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pvqz.g94 "https://www.basissetexchange.org/api/basis/cc-pvqz/format/gaussian94/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pv5z.g94 "https://www.basissetexchange.org/api/basis/cc-pv5z/format/gaussian94/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O cc-pwcvdz-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pwcvdz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O cc-pwcvtz-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pwcvtz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O cc-pwcvqz-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pwcvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O cc-pwcv5z-pp.g94 "https://www.basissetexchange.org/api/basis/cc-pwcv5z-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pcvdz.g94 "https://www.basissetexchange.org/api/basis/cc-pcvdz/format/gaussian94/?version=0&elements=3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,20&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pcvtz.g94 "https://www.basissetexchange.org/api/basis/cc-pcvtz/format/gaussian94/?version=0&elements=3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,20&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O cc-pcvqz.g94 "https://www.basissetexchange.org/api/basis/cc-pcvqz/format/gaussian94/?version=0&elements=3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,20&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O myaug-cc-pvdz.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvdz/format/gaussian94/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O myaug-cc-pvtz.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvtz/format/gaussian94/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O myaug-cc-pvqz.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz/format/gaussian94/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O myaug-cc-pv5z.g94 "https://www.basissetexchange.org/api/basis/aug-cc-pv5z/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O myaug-cc-pv6z.g94 "https://www.basissetexchange.org/api/basis/aug-cc-pv6z/format/nwchem/?version=1&elements=1,2,5,6,7,8,9,10,13,14,15,16,17,18&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pwcvdz-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pwcvtz-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pwcvqz-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pwcv5z-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pvdz-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pvtz-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pvqz-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget   --progress=dot:giga -O  myaug-cc-pv5z-pp.g94 "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/gaussian94/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&& chmod +x /opt/ecp_strip.sh \
&& /opt/ecp_strip.sh cc-pvdz-pp.g94  \
&& /opt/ecp_strip.sh cc-pvtz-pp.g94  \
&& /opt/ecp_strip.sh cc-pvqz-pp.g94  \
&& /opt/ecp_strip.sh cc-pv5z-pp.g94  \
&& /opt/ecp_strip.sh cc-pwcvdz-pp.g94  \
&& /opt/ecp_strip.sh cc-pwcvtz-pp.g94  \
&& /opt/ecp_strip.sh cc-pwcvqz-pp.g94  \
&& /opt/ecp_strip.sh cc-pwcv5z-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pvdz-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pvtz-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pvqz-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pv5z-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pwcvdz-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pwcvtz-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pwcvqz-pp.g94  \
&& /opt/ecp_strip.sh myaug-cc-pwcv5z-pp.g94  \
&&           wget  --progress=dot:giga -O  cc-pvdz-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pvdz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&&           wget  --progress=dot:giga -O  cc-pvtz-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pvtz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&&           wget  --progress=dot:giga -O  cc-pvqz-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&&           wget  --progress=dot:giga -O  cc-pv5z-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pv5z-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&&           wget  --progress=dot:giga -O  cc-pwcvdz-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pwcvdz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&&           wget  --progress=dot:giga -O  cc-pwcvtz-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pwcvtz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&&           wget  --progress=dot:giga -O  cc-pwcvqz-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pwcvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&&           wget  --progress=dot:giga -O  cc-pwcv5z-pp.ecp "https://www.basissetexchange.org/api/basis/cc-pwcv5z-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
&& 	     wget  --progress=dot:giga -O  myaug-cc-pwcvdz-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O  myaug-cc-pwcvtz-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O  myaug-cc-pwcvqz-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O  myaug-cc-pwcv5z-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pwcvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O  myaug-cc-pvdz-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O  myaug-cc-pvtz-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O  myaug-cc-pvqz-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O  myaug-cc-pv5z-pp.ecp "https://www.basissetexchange.org/download_basis/basis/aug-cc-pvqz-pp/format/nwchem/?version=0&elements=29,30,31,32,33,34,35,36,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-svp.ecp     "https://www.basissetexchange.org/api/basis/def2-svp/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-svpd.ecp    "https://www.basissetexchange.org/api/basis/def2-svpd/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-tzvp.ecp    "https://www.basissetexchange.org/api/basis/def2-tzvp/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-tzvpp.ecp   "https://www.basissetexchange.org/api/basis/def2-tzvpp/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-tzvppd.ecp  "https://www.basissetexchange.org/api/basis/def2-tzvppd/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-tzvpd.ecp   "https://www.basissetexchange.org/api/basis/def2-tzvpd/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-qzvp.ecp    "https://www.basissetexchange.org/api/basis/def2-qzvp/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-qzvpp.ecp   "https://www.basissetexchange.org/api/basis/def2-qzvpp/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-qzvppd.ecp  "https://www.basissetexchange.org/api/basis/def2-qzvppd/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57&optimize_general=true&uncontract_general=true" \
	    && wget  --progress=dot:giga -O def2-qzvpd.ecp   "https://www.basissetexchange.org/api/basis/def2-qzvpd/format/nwchem/?version=1&elements=1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,57&optimize_general=true&uncontract_general=true" \
&& chmod +x /opt/ao_strip.sh \
&& /opt/ao_strip.sh cc-pvdz-pp.ecp \
&& /opt/ao_strip.sh cc-pvtz-pp.ecp \
&& /opt/ao_strip.sh cc-pvqz-pp.ecp \
&& /opt/ao_strip.sh cc-pv5z-pp.ecp \
&& /opt/ao_strip.sh cc-pwcvdz-pp.ecp \
&& /opt/ao_strip.sh cc-pwcvtz-pp.ecp \
&& /opt/ao_strip.sh cc-pwcvqz-pp.ecp \
&& /opt/ao_strip.sh cc-pwcv5z-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pvdz-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pvtz-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pvqz-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pv5z-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pwcvdz-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pwcvtz-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pwcvqz-pp.ecp \
&& /opt/ao_strip.sh myaug-cc-pwcv5z-pp.ecp  \
&& /opt/ao_strip.sh def2-svp.ecp \
&& /opt/ao_strip.sh def2-svpd.ecp \
&& /opt/ao_strip.sh def2-tzvp.ecp \
&& /opt/ao_strip.sh def2-tzvpp.ecp \
&& /opt/ao_strip.sh def2-tzvpd.ecp \
&& /opt/ao_strip.sh def2-tzvppd.ecp \
&& /opt/ao_strip.sh def2-qzvp.ecp \
&& /opt/ao_strip.sh def2-qzvpp.ecp \
&& /opt/ao_strip.sh def2-qzvpd.ecp \
&& /opt/ao_strip.sh def2-qzvppd.ecp \
	    && echo 'caching libint' \
	    && cd /opt/install/exachem/ \
	    && if [[ $FETCHED_LIBINT == 1 ]]; then rm -f /opt/cache/libint.tar.bz2 || true; else tar cjvf /opt/cache/libint.tar.bz2 ./include/libint* ./share/libint* ./lib/pkgconfig/libint*.pc ./lib/cmake/libint* ./lib/libint*  || true ; fi  \
            && echo 'clean unnecessary source to reduce docker size' \
            && cd /opt && rm -rf exachem \
	    && cd /usr/local || { echo "Failure"; exit 1; } ; if [[ $FETCHED_MPICH == 1 ]]; then rm -f /opt/cache/mpich.tar.bz2 || true; else tar cjvf /opt/cache/mpich.tar.bz2 lib/* include/* bin/* share/* ||true ; fi\
            && echo 'clean unnecessary packages ' \
            && DEBIAN_FRONTEND=noninteractive apt-get -q=2 -y install localepurge \
	    && echo MANDELETE >  /etc/locale.nopurge \
	    && echo DONTBOTHERNEWLOCALE >> /etc/locale.nopurge \
	    && echo en_US.UTF-8 >> /etc/locale.nopurge \
	    && localepurge \
	    && rm -rf /usr/share/doc \
            &&  apt-get -q=2 -y purge   patch wget  make git cpp-"$GCC_VERSION" gfortran-"$GCC_VERSION" gcc-"$GCC_VERSION" libicu-dev libpython"$PY_VERSION"-dev libstdc++-"$GCC_VERSION"-dev libc6-dev libgcc-"$GCC_VERSION"-dev libgfortran-"$GCC_VERSION"-dev  || true \
            && apt-get -q=2 -y install libgfortran5 libgomp1 && apt-get clean \
	    && apt-get purge -y cuda-cccl-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libcublas-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libcufft-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libcufft-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libcusolver-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libcusparse-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libnpp-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" libcurand-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" cuda-nvrtc-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR"  libcufile-dev-"$CUDA_VERSION_MAJOR"-"$CUDA_VERSION_MINOR" || true ; rm -f /usr/local/cuda-*/targets/x86_64-linux/lib/*static*a || true \
&& rm -rf /opt/TAMM  \
&& rm -rf /opt/exachem/build \
&& rm -rf /opt/install/exachem/bin/xc-info /opt/install/exachem/bin/h5* \
&& rm -rf /opt/install/exachem/lib/lib*a \
&& rm -rf /cmake* \
&& du -sk /opt/*|sort -n \
&& echo "dddddddddddd"  \
&& dpkg-query -Wf '${Installed-Size}\t${Package}\n' | sort -n | tail -n 50 \
&& echo "dddddddddddd" \
	    && df -h \
	    && echo '*** ls -l CC/bin **' \
	    && ls -l /opt/install/exachem/bin/ || true \
	    && echo '*** ldd ***'  \
	    && LD_LIBRARY_PATH=/usr/local/lib:/opt/lib ldd /opt/install/exachem/bin/ExaChem || true \
	    && dpkg-query -Wf '${Installed-Size}\t${Package}\n' | sort -n | tail -n 100
ENV	    CUDA_VERSION_MAJOR=11 
ENV	    CUDA_VERSION_MINOR=7
ENV	    LD_LIBRARY_PATH=/usr/local/cuda-"$CUDA_VERSION_MAJOR"."$CUDA_VERSION_MINOR"/lib64:/opt/lib \
	    PATH=/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/cuda-"$CUDA_VERSION_MAJOR"."$CUDA_VERSION_MINOR"/bin:/opt/install/exachem/bin:/usr/local/bin
#clean unnecessary packages
#RUN         apt-get -y remove  make  unzip cmake   &&  apt-get -y autoremove && apt-get clean

USER        "$MYUSER"

#WORKDIR     /data
#ENTRYPOINT  ["mpirun","-np","3","nwchem"]
