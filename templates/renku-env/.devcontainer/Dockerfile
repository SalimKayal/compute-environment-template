FROM docker.io/r-base:latest as r

USER root
# Generally, Dev Container Features assume that the non-root user (in this case jovyan)
# is in a group with the same name (in this case jovyan). So we must first make that so.
RUN groupadd user \
    && usermod -g user -a -G staff,users user

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      tini \
    && rm -rf /var/lib/apt/lists/*

USER user

RUN R -e "install.packages('renv', repos = c(CRAN = 'https://cloud.r-project.org'))"

WORKDIR /home/user/workdir/

COPY renv.lock* .devcontainer/noop.txt /home/workdir/
COPY app.R* .devcontainer/noop.txt /home/workdir/
COPY .Rprofile* .devcontainer/noop.txt /home/workdir/
COPY .Rprofile* .devcontainer/noop.txt /home/workdir/
COPY renv/activate.R* .devcontainer/noop.txt /home/workdir/renv/
COPY renv/settings.json* .devcontainer/noop.txt /home/workdir/renv/

RUN R -e "renv::restore()"

FROM quay.io/condaforge/miniforge3:latest as python

USER root
# Generally, Dev Container Features assume that the non-root user (in this case jovyan)
# is in a group with the same name (in this case jovyan). So we must first make that so.
RUN groupadd user \
    && usermod -g user -a -G users user

ENV CONDA_ENVS_DIRS /work/envs
ENV SHELL=/bin/bash

# Copy environment.yml (if found) to a temp location so we update the environment. Also
# copy "noop.txt" so the COPY instruction does not fail if no environment.yml exists.
COPY environment.yml* .devcontainer/noop.txt /tmp/conda-tmp/

USER user
RUN if [ -f "/tmp/conda-tmp/environment.yml" ]; then umask 0002 && \
    /opt/conda/bin/conda env create -n project -f /tmp/conda-tmp/environment.yml; fi && \
    conda init && \
    conda install -y -n project ipykernel && \
    /opt/conda/envs/project/bin/python -m ipykernel install --name project --display-name "Project environment" --user && \
    echo "conda activate project" >> $HOME/.bashrc

FROM r as universal

COPY --from=python /opt/conda /opt/conda

FROM ${templateOption:baseLanguage}

ENTRYPOINT ["tini", "--"]
CMD ["/bin/bash"]
