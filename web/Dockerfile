# base image to start with (Debian Bullseye slimmed down)
FROM python:3.11.2-slim-bullseye AS builder

# update the base image
RUN apt-get update && \
    apt-get upgrade --yes
    
# create a regular user to avoid malicious attacks on host machine 
# (Docker Containers initially built with superuser access)
RUN useradd --create-home macchi
USER macchi
WORKDIR /home/macchi

# create a new virtual environment for this image
# resetting the PATH to the virtual environment ensures persistence of the environment
ENV VIRTUALENV=/home/macchi/venv 
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

# upgrade pip/setuptools and install dependencies
# no need for caching of dependencies outside the virtual environment
COPY --chown=macchi pyproject.toml constraints.txt ./
RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir -c constraints.txt ".[dev]"

# install source files and tests
COPY --chown=macchi src/ src/
COPY --chown=macchi test/ test/

# run tests and checks as part of image build
RUN python -m pip install . -c constraints.txt && \
    python -m pytest test/unit/ && \
    python -m flake8 src/ && \
    python -m isort src/ --check && \
    python -m black src/ --check --quiet && \
    python -m pylint src/ --disable=C0114,C0116,R1705 && \
    python -m bandit -r src/ --quiet && \
    python -m pip wheel --wheel-dir dist/ . -c constraints.txt
# last command publishes the build package into dist folder

# stage 2
FROM python:3.11.2-slim-bullseye

RUN apt-get update && \
    apt-get upgrade --yes

RUN useradd --create-home macchi
USER macchi
WORKDIR /home/macchi

ENV VIRTUALENV=/home/macchi/venv 
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

# make a copy of the wheel file from the build stage
COPY --from=builder /home/macchi/dist/page_tracker*.whl /home/macchi/

RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir page_tracker*.whl

CMD ["flask", "--app", "page_tracker.app", "run", \
     "--host", "0.0.0.0", "--port", "5000"]