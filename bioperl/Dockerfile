############################################################
# Dockerfile to build a BioPerl container image
# Based on Ubuntu
############################################################

# Set the base image to the BioPerl prebuilt prerequisites image
FROM bioperl/bioperl-deps as builder

# File Author / Maintainer
LABEL maintainer="Hilmar Lapp <hlapp@drycafe.net>"

# Set the working directory to where we will install bioperl
# in the builder stage
WORKDIR /bioperl

# Prevent error messages from debconf about non-interactive frontend
ARG TERM=linux
ARG DEBIAN_FRONTEND=noninteractive

# Install software needed to build BioPerl (not all are required to run it)
RUN apt-get update --yes && \
    apt-get install --yes git
RUN cpanm -n Dist::Zilla Dist::Zilla::Plugin::Git

# download BioPerl soure, install authordeps, build BioPerl
RUN \
  curl -L -s -o - \
         https://github.com/bioperl/bioperl-live/archive/master.tar.gz | \
    tar xzf - && \
  cd bioperl-live-master && \
  dzil authordeps | cpanm -n && \
  dzil build --in /bioperl/bioperl-build

# Set the base image to the BioPerl prebuilt prerequisites image
# this time for the final image
FROM bioperl/bioperl-deps as final

# Install modules recommended by BioPerl.
# We can't include Bio::ASN1::EntrezGene here yet, because it declares
# a dependency on BioPerl, thus pulling in BioPerl first.
RUN cpanm -n \
  Bio::Phylo

# -------------------------------------------------------------
# Install BioPerl from GitHub current master branch.
#
# This is the actual installation step of BioPerl itself :-)
# -------------------------------------------------------------
# Copy the files produced in the previous build stage over to the 
# final image
COPY --from=builder /bioperl/bioperl-build /usr/local/src/bioperl-build/

# Install bioperl
RUN cpanm -n /usr/local/src/bioperl-build

# Install modules recommended by BioPerl that depend on BioPerl.
# (Don't ask. See above.)
RUN cpanm \
  Bio::ASN1::EntrezGene
