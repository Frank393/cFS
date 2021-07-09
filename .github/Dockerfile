FROM ubuntu:18.04

MAINTAINER Frank Gonzalez <frank.gonzalez3939@gmail.com>

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install build-essential -y

# Basic dependencies
RUN apt-get install -y git make sudo

# dependencies to install git from source
RUN apt-get install -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext cmake gcc
RUN curl -o git.tar.gz https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.26.2.tar.gz
RUN tar -zxf git.tar.gz
RUN cd git-* && make prefix=/usr/local && make prefix=/usr/local install
RUN apt-get purge -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext gcc

RUN git --version
# Install dependencies for build-cfs
RUN apt-get install -y lcov

# Install dependencies for build-documentation
RUN apt-get install -y doxygen graphviz texlive-latex-base \
      texlive-fonts-recommended texlive-fonts-extra texlive-latex-extra

# Install dependencies for static-analysis
RUN apt-get install -y cppcheck
